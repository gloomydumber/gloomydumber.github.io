+++
title = "Breaking Down Solidity Calls: call, staticcall, delegatecall"
date = 2022-06-25
[extra]
toc = true
comment = true
+++

_Upgradable Contract_ 에 대한 학습 과정에서, 이를 온전히 이해하기 위해서는 Solidity의 함수 호출에 관해서 선행적으로 정리할 필요가 있다고 판단했다. 따라서 본 글에서는 Solidity의 함수 호출 메소드 중 `call`, `staticcall`, `delegatecall` 을 다루고, 아울러 High-Level Call 과 Low-Level Call 의 비교도 함께 설명한다.

## Solidity Calls

우선 `call`, `staticcall`, `delegatecall` 모두 `address` 타입의 멤버 메소드이다. 아래와 같은 식으로 호출이 가능하다는 얘기이다.

```solidity
address(0xdAC17F958D2ee523a2206206994597C13D831ec7).call(...);
address(0xdAC17F958D2ee523a2206206994597C13D831ec7).staticcall(...);
address(0xdAC17F958D2ee523a2206206994597C13D831ec7).delegatecall(...);
```

[Solidity Docs | Members of Address Types](https://docs.soliditylang.org/en/v0.8.30/units-and-global-variables.html#members-of-address-types)에서 `balance`, `transfer`, `send` 등 `address` 타입의 모든 멤버에 대한 정의와 동작을 확인해 볼 수 있다.

[Solidity Docs | Members of Addresses](https://docs.soliditylang.org/en/v0.8.30/types.html#members-of-addresses)에서는 `call`, `staticcall`, `delegatecall` 에 대해 다음과 같이 언급하고 있다.

> In order to interface with contracts that do not adhere to the ABI, or to get more direct control over the encoding, the functions `call`, `delegatecall` and `staticcall` are provided. They all take a single `bytes memory` parameter and return the success condition (as a `bool`) and the returned data (`bytes memory`). The functions `abi.encode`, `abi.encodePacked`, `abi.encodeWithSelector` and `abi.encodeWithSignature` can be used to encode structured data.

이에 따르면, `call`, `staticcall`, `delegatecall` 은 컨트랙트를 ABI(Application Binary Interface)에 관계없이 직접적으로 제어하고자 할 때 사용하는 함수(메소드)들이다. 여기서 **contracts that do not adhere to the ABI** 라는 표현의 뉘앙스가 조금 이해하기 어렵다. 직역했을 때 **ABI를 따르지 않는** 혹은 **ABI를 고수하지 않는** 컨트랙트란, **ABI를 통한 호출이 불가능한 컨트랙트** 라는 뜻이다. 일반적으로 컨트랙트를 Solidity나 Vypher를 통해 작성하고 컴파일하는 경우에 자연스레 ABI가 함께 생성되며, 컨트랙트를 직접 raw-level의 바이트코드로 작성하는 경우 등 아주 특이한 경우에만 ABI를 통한 컨트랙트 호출이 불가능하다. (혹자는 바이트코드로 컨트랙트를 작성하는게 특이하지 않다고 할 수도 있다)

{% detail(title="일반적인 상황이 아니라서, 크게 고려하지 않아도 되지만 다음과 같은 컨트랙트 코드가 ABI를 통한 호출이 불가능한 컨트랙트의 예이다.") %}

아래 컨트랙트는 동작이 별도의 함수로 정의되어 있지않고 `fallback()`으로만 구성되어, ABI를 통한 호출이 불가능하다.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract NonABIContract {
    uint256 public storedNumber;
    address public storedAddress;

    // No function selector, just raw data parsing
    fallback() external payable {
        // Expect calldata layout:
        // [0..31]   = uint256 number
        // [32..51]  = address (left-padded to 32 bytes)

        require(msg.data.length >= 52, "Invalid calldata");

        uint256 number;
        address addr;

        // Load the first 32 bytes as uint256
        assembly {
            number := calldataload(0)
        }

        // Load the next 32 bytes, but only the last 20 are used for address
        bytes32 rawAddr;
        assembly {
            rawAddr := calldataload(32)
        }
        addr = address(uint160(uint256(rawAddr)));

        storedNumber = number;
        storedAddress = addr;
    }
}
```

위의 컨트랙트 로직은 `fallback()` 에서만 동작하므로 `interface` 키워드로 선언한 `INonABIContract` 에서 정의된 것이 없다. 결국 아래 코드 예제에서 `call` 을 이용해 접근하고 있다.

```solidity ,hl_lines=20-21
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface INonABIContract {
    // No functions here, because the contract does not expose an ABI.
    // We'll just use raw call.
}

contract Caller {
    function callNonABIContract(
        address target,
        uint256 number,
        address addr
    ) external returns (bool success, bytes memory returnData) {
        // Manually encode the calldata:
        // [0..31] = uint256 number
        // [32..51] = address (20 bytes, left padded to 32)
        bytes memory payload = abi.encodePacked(number, addr);

        // Use low-level call
        (success, returnData) = target.call(payload);

        require(success, "Low-level call failed");
    }
}
```

이렇듯, 아주 특수한 경우에만 컨트랙트가 ABI를 통해 호출이 불가능하다.

{% end %}

정리하자면, **`call`, `staticcall`, `delegatecall` 은 ABI를 통한 호출이 불가하거나, 인코딩을 통해 보다 직접적인 제어가 필요한 컨트랙트를 호출하기 위해 사용하는 함수다.**

또, `call`, `staticcall`, `delegatecall` 모두 파라미터로는 `bytes memory` 타입의 하나의 값을 취하고,

반환값으로는 성공 여부(`bool`)와 반환 데이터(`bytes memory`) 두 값을 반환한다.

이를 코드 예제로 확인해보면,

```solidity ,hl_lines=2
bytes memory payload = abi.encodeWithSignature("register(string)", "MyName");
(bool success, bytes memory returnData) = address(nameReg).call(payload);
require(success);
```

2번째 줄의 `call` 내부에 `bytes memory` 타입의 `payload` 변수 하나가 파라미터로 전달되었고,

2번째 줄의 `(bool success, bytes memory returnData)` 로 `()` 를 통해 반환값으로 두 변수가 각각 `bool`과 `bytes memory` 타입으로 반환되는 것을 확인할 수 있다.

이에 더해, 문서에서 해당 내용 바로 아래에 다음과 같은 경고 Callout을 두었는데,

{% caution(title="Warning") %} All these functions are low-level functions and should be used with care. Specifically, any unknown contract might be malicious and if you call it, you hand over control to that contract which could in turn call back into your contract, so be prepared for changes to your state variables when the call returns. The regular way to interact with other contracts is to call a function on a contract object (`x.f()`). {% end %}

즉, `call`, `staticcall`, `delegatecall` 모두 공통적으로 **low-level function** 이므로 조심해서 사용해야한다고 언급하고 있다. 특히, 알려지지 않은 컨트랙트를 호출 할 때에는 제어권을 해당 컨트랙트에 넘겨주기 때문에 주의해야한다고 언급한다. 따라서 다른 컨트랙트의 함수를 호출할 때에는 일반적으로 `x.f()` 와 같은 방법으로 컨트랙트 객체를 이용하여 접근하는 방식을 택하는 것을 권장하고 있다.

이로써 `call`, `staticcall`, `delegatecall`의 공통점을 요약하면,

- `address` 타입의 멤버 메소드이다.
- ABI를 통해 호출이 불가능하거나, 인코딩을 통해 보다 직접적인 제어가 필요한 컨트랙트를 호출하기 위해 사용하는 메소드이다.
- 실행 시, `bytes memory` 타입의 변수 하나를 파라미터로 취하고, 실행 결과 반환값으로는 `bool`, `bytes memory` 변수 2개를 반환받는다.
- **low-level function** 이므로 보안을 고려해서 사용해야한다.

이쯤에서 **low-level function** 이라는 용어에 대해서 더 알아보려고 한다.

## High-Level Call vs. Low-Level Call

컨트랙트의 함수 인터페이스를 알면 `call`, `staticcall`, `delegatecall` 을 사용할 필요없이 아래와 같이 호출 할 수 있다.

```solidity ,hl_lines=18
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CalledContract {
    uint256 public number;

    function setNumber(uint256 _num) external {
        number = _num;
    }
}

interface ICalledContract {
    function setNumber(uint256 _num) external;
}

contract CallerContract {
    function callSetNumber(address _callee, uint256 _num) external {
        ICalledContract(_callee).setNumber(_num);
    }
}
```

`CalledContract` 의 인터페이스가 `ICalledContract` 로 `interface` 키워드를 통해 `setNumber` 함수와 함께 정의되어 있으므로, `CallerContract` 에서 `.setNumber()` 와 같이 멤버 메소드를 호출하는 것과 같은 형태로 호출이 가능하다. 이러한 방식의 호출을 **High-Level Call** 이라고 한다.

High-Level Call 은 아래와 같은 동작을 한다.

- 함수 이름과 파라미터를 인터페이스로 정의하고 호출하면 컴파일러가 함수 이름과 파라미터를 ABI에 맞게 알아서 인코딩해준다.
- 실패 시에는 트랜잭션 자체가 `revert` 된다.

반면에 아래와 같이 `call` 을 활용해서 호출하는 경우를 살펴보자.

```solidity ,hl_lines=14-17
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CalledContract {
    uint256 public number;

    function setNumber(uint256 _num) external {
        number = _num;
    }
}

contract CallerContract {
    function callSetNumber(address _callee, uint256 _num) external returns (bool, bytes memory) {
        // encode function selector + arguments
        (bool success, bytes memory data) = _callee.call(
            abi.encodeWithSignature("setNumber(uint256)", _num)
        );
        return (success, data);
    }
}
```

이 경우, `interface` 키워드를 통해 작성해 둔 인터페이스가 없으므로, 개발자가 직접 `abi.encodeWithSignature` 코드를 통해 함수 이름과 파라미터를 인코딩하고 있다.

이렇게 인코딩된 값을 `call` 을 이용하여 **직접** 호출하는 것을 **Low-level Call**이라 한다.

`call`, `staticall`, `delegatecall` 과 같은 Low-Level Call 은 아래와 같은 동작을 한다.

- 개발자가 직접 함수 시그니처와 파라미터를 `abi.encode`와 같은 메소드를 통해 인코딩해서 `bytes memory`로 넘겨줘야 한다.
- 실패시 트랜잭션 자체가 `revert` 되지 않고, 성공/실패 여부도 `bool` 타입으로 개발자가 직접 예외 처리해야 한다. (Silently Fail 될 수 있음을 고려해야 한다)

## `call`, `staticcall`, `delegatecall`

다시 한번 더 정리하면 Low-level Call에 해당하는 세가지 메소드의 공통된 특징은 아래와 같다.

- `address` 타입의 멤버 메소드이다.
- ABI를 통해 호출이 불가능하거나, 인코딩을 통해 보다 직접적인 제어가 필요한 컨트랙트를 호출하기 위해 사용하는 메소드이다.
- 실행 시, `bytes memory` 타입의 변수 하나를 파라미터로 취하고, 실행 결과 반환값으로는 `bool`, `bytes memory` 변수 2개를 반환받는다.
- **low-level function** 이므로 보안을 고려해서 사용해야한다.
- 개발자가 직접 함수 시그니처와 파라미터를 `abi.encode`와 같은 메소드를 통해 인코딩해서 `bytes memory`로 넘겨줘야 한다.
- 실패시 트랜잭션 자체가 `revert` 되지 않고, 성공/실패 여부도 `bool` 타입으로 개발자가 직접 예외 처리해야 한다. (Silently Fail 될 수 있음을 고려해야 한다)

아래에서는 Low-level Call에 해당하는 세가지 메소드 각각의 특징에 대해서 간략하게 서술한다.

### `call`

- `address(...).call{value: 1 ether, gas: 10000}("");` 와 같이 `bytes memory` 파라미터를 비워두면 함수 호출이 아니라 이더 전송으로 쓸 수 있고, 보낼 이더 수량과 가스를 지정할 수 있다. (이는 이더 전송시 `send`와 달리 권장되는 방법이다)
- 다른 컨트랙트의 함수 호출이 가능하다. (하지만 High-level Call을 권장한다)
- 호출하는 컨트랙트(callee)에 상태 변경 로직이 있으면 callee의 상태가 변경된다.

### `staticcall`

- read-only 버전의 `call`로 이해할 수 있다.
- 컨트랙트의 `view` 또는 `pure` modifier인 함수를 호출할 때 사용한다.
- 상태변경이나 상태저장이 발생하면 `revert` 된다.

### `delegatecall`

- 다른 컨트랙트(callee)의 로직을 이용하되, 스토리지는 호출자(caller)의 것을 이용한다.
- Upgradable Contract를 구현할 때 이용한다.
- Storage Collision이 일어나지 않도록 주의해서 사용해야 한다.

---

- [Solidity Docs | members of address types](https://docs.soliditylang.org/en/v0.8.30/units-and-global-variables.html#members-of-address-types)
- [RareSkills | Delegatecall: The Detailed and Animated Guide](https://rareskills.io/post/delegatecall)
- [RareSkills | Low Level Call vs High Level Call in Solidity](https://rareskills.io/post/low-level-call-solidity)
- [Coinmonks | Learn Solidity lesson 34. Call, staticcall and delegatecall](https://medium.com/coinmonks/call-staticcall-and-delegatecall-1f0e1853340)

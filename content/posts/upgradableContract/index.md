+++
title = "Upgradable Smart Contracts"
date = 2022-06-26
[extra]
toc = true
comment = true
+++

# Upgradable Smart Contracts

해당 포스트는 *moralis*의 *What are Upgradable Smart Contracts? Full Guide*의 번역 및 요약입니다.

## What are Upgradable Smart Contracts? Full Guide

|    ![01](img/01.png)     |
| :-----------------------------------------------------------------: |
| <b>Upgradable Smart Contract</b> |

능숙한 Smart Contract 개발자들은 컨트랙트를 좀 더 나은 코드로 향상시키기 위해서 고군분투한다.
그러기 위해서 Smart Contract를 수정하거나, 업그레이드한다. 그러나, *업그레이드 가능한 스마트 컨트랙트*의 개념은 경험있는 개발자에게도 이해하기에 어려움이있다.
즉, _업그레이드 가능한 스마트 컨트랙트_ 개념은 초보 블록체인 개발자의 범주에 있는 것이 아니다. 그럼에도, 이 포스팅은 독자가 초보자라도 굉장히 유용할 것이다.
그래서, *업그레이드 가능한 스마트 컨트랙트*가 대체 뭘까? 다음의 원문을 읽고 *업그레이드 가능한 스마트 컨트랙트*의 근본이 무엇이고 어떻게 동작하는지 알아보자.

계속해서, 독자는 먼저 *업그레이드 가능한 스마트 컨트랙트*가 무엇인지 배우고, 어떻게 기능하는지 알아볼 것이다.
그러고나서 *transparent proxy*와 *upgradable proxy*의 차이에 대해 배울 것이다.
추가로, 우리는 *업그레이드 가능한 스마트 컨트랙트*를 어떻게 만드는지 설명하고, 이 *업그레이드 가능한 스마트 컨트랙트*가 어떤 제약을 해결할 수 있는지 알아볼 것이다.
또, 예제 프로젝트를 진행할 기회도 얻을 수 있을 것이다. 결과적으로는 NFT 민팅 DApp을 구성하는 데에도 활용할 수 있을 것이다.
후자는 독자가 추후에 업그레이드 할 스마트 컨트랙트 포함하는 프로젝트이다. 그러므로, 독자는 *업그레이드 가능한 스마트 컨트랙트*가 어떻게 동작하는지 적절하게 이해할 수 있을 것이다.

언급한대로, *업그레이드 가능한 스마트 컨트랙트*는 처음에는 이해하기에 난해한 부분이 있다. 그러므로, 블록체인 개발의 기본을 어느정도 알고나서 이 개념을 공부하는 것을 추천한다.
이 포스팅을 읽기 전에, _"Web3는 어떻게 동작하는가?"_, _"Solidity란 무엇인가?"_, _"Smart Contracts란 무엇인가?"_ 와 같은 포스팅을 읽고 이해했으면 한다.

## What are Upgradable Smart Contracts?

독자는 아마도 이더리움과 같은 프로그램 가능한 블록체인에서, 스마트 컨트랙트는 아주 핵심적인 부분인 것을 이해하고 있을 것이다. 사전에 정의된 규칙에 따라서 작업이 실행되도록 함으로써, 스마트 컨트랙트는 순서를 지키며 동작한다. 스마트 컨트랙트가 없다면, *token*이나 _NFT_, _DApp_ 또한 존재할 수 없다. 그렇다면 *업그레이드 가능한 스마트 컨트랙트*란 대체 무엇일까? 일단 주지해야하는 사실이 있다. *"업그레이드 가능한"*이라는게, 변할수 있다(mutable)는 개념을 뜻하는게 아니다. *EVM*의 기본 규칙중 하나는, 일단 스마트 컨트랙트가 블록체인 상에 deploy 되는 순간, 변할 수 없다. 즉, immutable(불변적)이다. 이러한 *EVM*의 기본 규칙을 어길 수 없기 때문에, *업그레이드 가능한 스마트 컨트랙트*는 특수한 프록시 패턴을 사용한다. 구체적으로, *proxy contract*와 *implementation contract(logic contract)*를 별도로 배포하는 식이다.

![02](img/02.png)

## How do Upgradable Smart Contracts Work?

위의 도식을 보면, 우저가 logic contract를 proxy contract를 거쳐서 상호작용하고 있음을 알 수 있다. 이것이 가능한 것은, proxy contract가 logic contract의 주소를 저장할 수 있기 때문이다. 만약 우리가 새로운 logic contract를 구현해서 deploy했다면, proxy contract에 저장된 logic contract의 주소를 바꿔주기만 하면 된다.

*업그레이드 가능한 스마트 컨트랙트*를 구현하기 위한 프록시 패턴에는 여러 방법이 있다. 다만, 프록시 패턴이 여러 유형으로 존재하지만, 대부분의 프록시 패턴은 *transparent proxy*와 *UUPS(Universal Upgradable Proxy Standard)*의 형태로 구현된다. 다행히도, 이 두 타입의 예제 코드 모두 *OpenZeppelin*에서 제공하고있다.

![03](img/03.png)

## Transparent vs UUPS Proxy

앞서 언급한 바와 같이, *transparent*와 _UUPS_ 두 가지의 *proxy pattern*들은 가장 보편적인 방법이다. 두 방법 모두 같은 원리를 사용하지만, 약간의 다른 구조를 취하고 있다. 이에 둘을 비교하고, 두 프록시 패턴의 주요한 특징을 알아보도록하자.

**Transparent Proxy Pattern Type:**

- 업그레이드가 Proxy Contract에서 다뤄짐
- 상대적으로 고비용의 Deployment
- 상대적으로 유지보수가 쉬움

**UUPS Proxy Pattern Type:**

- 업그레이드가 implementation contract에서 다뤄짐 (_업그레이드에 관한 기능, 함수를 implementation contract 상에서 구현해야하는 것을 의미_)
- 상대적으로 저비용의 Deployment
- 상대적으로 유지보수가 어려움

![04](img/04.png)

## How to Make a Smart Contract Upgradable?

이 지점에서, *업그레이드 가능한 스마트 컨트랙트*가 무엇인지, 어떻게 동작하는지 이해하고, 두개의 주요한 proxy pattern 유형에 대해서 알아보았다. 그러므로, 독자는 이제 *업그레이드 가능한 스마트 컨트랙트*를 구성하기에 준비되었다 할 수 있다. 게다가, 앞서 언급했듯이 우리는 처음부터 코드를 다 짤필요가 없다. *Openzepplin*의 implementation을 사용할 수 있기 때문이다. 즉, 우리는 프록시 패턴을 우리 스스로 구현할 필요는 없다는 의미다. 이하는 *업그레이드 가능한 스마트 컨트랙트*를 구성하기 위한 단계들이다.

1. 먼저, 우리는 _initializable_ 컨트랙트를 상속해야한다.

```solidity
contract ExampleCOntractName is initializable {}
```

위는 default로는 *transparent proxy pattern*이다. 만약 *UUPS proxy pattern*을 사용한다면 아래와 같이 _UUPSUpgradable_ 상속을 추가해야한다.

```solidity
contract ExampleContractName is initializable, UUPSUpgradable {}
```

2. 다음으로, *constructor*를 *initializer*로 바꾸어야할 필요가 있다. 즉, *constructor(){}*를 *function initialize() public initializer {}*로 바꾸어야한다. 함수 이름이 *initialize*일 필요는 없으나, 끝의 _initializer {}_ 부분은 필수적이다.

3. 그리고, _OpenZeppelin_ 컨트랙트 라이브러리를 _upgradable_ 버전으로 바꿔줘야한다.

![05](img/05.png)

4. _initialize_ 함수에서, *upgradable contracts*의 함수인 *\_init*을 다음과 같이 호출해야한다.

```solidity
function initialize() initializer public {
__ERC1155_init(“”);
__Ownable_init();
__UUPSUpgradeable_init();
}
```

5. 마지막으로, *msg.sender*를 *\_msgSender()*로 대체한다. proxy address가 아닌, 유저의 wallet address와 상호작용해야하기 때문이다.

## The main Limitation of Upgradable Smart Contracts

주요한 *업그레이드 가능한 스마트 컨트랙트*의 한계점은, *storage collision*이다

![06](img/06.png)

위의 이미지에 따르면, 새로운 컨트랙트 버전에서 새로운 변수명이 위에 선언되어있고, *storage collision*을 일으키고 있는 것을 보여준다.

새로운 변수명은 꼭 아래 부분에 정의해야 *storage collison*이 일어나지 않는다.

## TL;DR

1. Immutable한 블록체인 환경에서도 *업그레이드 가능한 스마트 컨트랙트*를 배포할 수 있다.
2. 그를 위해서 *Proxy Pattern*을 사용하는데, *proxy contract*와 *implementation contract(logic contract)*를 별도로 배포하는 방식이다.
3. *Proxy Pattern*의 형태는 여러가지 형태가 있다.
4. _(본문에는 없지만)_ 구현 간에 *delegate call*을 이용한다.

## Yul Test

```yul
{
    let x := 0
    let y := add(x, 2)
    sstore(0, y)
}
```

```yul
{
    function addTwice(a, b) -> result {
        let sum := add(a, b)
        result := add(sum, sum)
    }

    let r := addTwice(3, 5)
    sstore(0, r)
}
```

```yul
{
    let n := calldataload(0)

    if eq(n, 0) {
        revert(0, 0)
    }

    switch n
    case 1 {
        sstore(0, 100)
    }
    case 2 {
        sstore(0, 200)
    }
    default {
        sstore(0, 999)
    }
}
```

```yul
{
    let size := calldatasize()
    let ptr := mload(0x40)
    calldatacopy(ptr, 0, size)
    return(ptr, size)
}
```

## References

- [post about upgradable Smart Contract on moralis.io](https://moralis.io/what-are-upgradable-smart-contracts-full-guide/)
- [youtube video about upgradable Smart Contract on moralis.io](https://www.youtube.com/watch?v=af1i0z0jhkg)
- [constructor 대신 initializer 함수를 사용하는 이유](https://stackoverflow.com/questions/72475214/solidity-why-use-initialize-function-instead-of-constructor)
- [medium post about delegate call](https://medium.com/coinmonks/delegatecall-calling-another-contract-function-in-solidity-b579f804178c)
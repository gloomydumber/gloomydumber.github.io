+++
title = "Arbitrum Sequencer Feed Validation with WebAssembly"
date = 2024-10-15
[extra]
toc = true
comment = true
+++

Sequence란 L2에서 처리 될 트랜잭션의 순서를 정렬하는 과정 또는 그러한 작업 그 자체를 뜻한다. L2 블록체인이 동작하는 데 있어서 일련의 트랜잭션의 순서를 조작하거나, 특정 트랜잭션을 악의적으로 생략하여 Front / Back Running, Sandwich Attack과 같은 MEV 공격이 일어날 수 있는 중요한 과정 중 하나이다. 이러한 Sequence 동작을 수행하는 주체를 Sequencer라 하는데, L2 중에서도 **Arbitrum**의 Sequencer는 자신이 수행하는 Sequence 결과를 두 가지 방법으로 공표(publish)한다.

첫 번째 방법으로는, 롤업의 과정 중의 하나이자 Sequence의 결과를 공표하는 일반적인 방법으로, Sequence 처리 된 트랜잭션의 순서를 포함한 정보를 일련의 데이터 압축 과정을 거쳐 Ethereum calldata 및 blob의 형태로 구성된 트랜잭션을 L1인 Ethereum에 제출하는 Batch Posting의 형태로 공표(publish)하는 방법이 있다. 만일 그렇게 제출된 트랜잭션에 잘못된 정보가 존재한다면, Fraud Proof라고 불리는 과정을 거치게 된다. Fraud Proof 과정에서는 잘못된 정보를 발견하여 제출하는 생태계 참여자에게 경제적 보상을 제공하는데, 이러한 경제적 유인을 통해 악의적 행동이나 예기치 못한 동작이 일어나지 않도록 항상 감시하게 한다.

두 번째 방법은 첫 번째 방법을 보조하는 개념이다. 실시간 피드를 발행하는 (publishing real-time feed) 방법인데, 실시간 피드는 누구나 구독하여 WebSocket 혹은 HTTP 스트림을 통해 메세지의 형태로 받아 볼 수 있다. 피드는 그 자체로서 Sequencer가 해당 트랜잭션을 특정한 순서로 처리할 것임을 공표하는 일종의 약속이라고 할 수 있고, 피드를 통해 트랜잭션 순서의 조작과 같은 중앙화 된 Sequencer의 악의적 행동이나 예기치 못한 동작 등, 피드의 내용과 실제 L2 네트워크에서 처리 된 트랜잭션이 정상적으로 서로 일치하는지 여부를 생태계 참여자 모두가 감시할 수 있다.

L2에서 말하는 Rollup의 핵심적인 처리는 전통적으로 첫 번째 방법으로 이루어지는 것이고, Arbitrum은 트랜잭션이 온체인에서 처리되기 직전에 Sequnecer Feed를 제공하는 두 번째 방법으로 이용자들에게 좀 더 투명성을 제공해주고자 한다. 첫 번째 방법이 필수적이고 이 방법을 통해서야 확실이 트랜잭션이 L1에 제출되었다는 **Hard Confirmation**을 얻을 수 있다. 두 번째 방법을 통해서는 아직 L1에서 확실한 보증이 이루어지진 못했지만 유저에게 좀 더 빨리 알려주는 식으로 **Soft Confirmation**이라는 개념으로 보증하는 것이다. Hard Confirmation이 이루어지는 기준으로 유저에게 온체인 정보와 UX를 제공하면 L1의 속도와 별다를바 없기 때문에, L2의 Soft Confirmation이라는 과정을 통해 약간의 보안성과 완결성을 포기하고 빠른 속도의 UX를 얻는 것이다.

이 글에서는 우선 Arbitrum Sequencer Feed가 실제 온체인의 데이터와 일치하는지 검증하는 과정에 대해 알아보고, 그 검증 과정을 Go 언어를 웹 어셈블리로 컴파일하여 웹 환경에서도 검증해 볼 수 있도록 하는 것에 대해 다룬다.

## Arbitrum Sequencer Feed Validation

공식 문서 [Arbitrum Docs | How to read the Sequencer Feed](https://docs.arbitrum.io/run-arbitrum-node/sequencer/read-sequencer-feed) 에서, Sequencer Feed를 읽는 방법을 잘 설명하고 있으므로 해당 내용에 기초하여 Sequencer Feed를 WebAssembly를 통해 웹 환경에서 검증하는 방법에 대해 알아보겠다.

Arbitrum Sequencer Feed는 WebSocket 메시지의 형태로 수신할 수 있으며, 수신되는 단위 메시지의 형태는 아래 예제의 JSON Format과 같다. 각 단위 메시지는, L2에서 처리된 하나의 블록에 대한 정보를 담고 있고, 특히 `l2Msg` 필드는 해당 블록에 포함된 트랜잭션들의 정보를 담고있다.

이러한 메시지 구조를 코드 레벨로 살펴보기 위해, 메시지가 어떻게 구성되어있는지 차근차근하게 살펴보겠다. 메시지 파싱을 위해 메시지 구성을 알아보는 것이지, 메시지 자체에 대해서 깊이있게 알 필요는 없으니, 구체적인 메시지 구성을 기억하기보다는 가볍게 살펴보며 과정을 따라가자. 이 예제 JSON 응답은 아래에서 계속 이루어질 파싱 과정에서 계속 활용될 예정이다.

```json
{
  "version": 1,
  "messages": [
    {
      "sequenceNumber": 25757171,
      "message": {
        "message": {
          "header": {
            "kind": 3,
            "sender": "0xa4b000000000000000000073657175656e636572",
            "blockNumber": 16238523,
            "timestamp": 1671691403,
            "requestId": null,
            "baseFeeL1": null
          },
          "l2Msg": "BAL40oKksUiElQL5AISg7rsAgxb6o5SZbYNoIF2DTixsqDpD2xII9GJLG4C4ZAhh6N0AAAAAAAAAAAAAAAC7EQiq1R1VYgL3/oXgvD921hYRyAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAArAAaAkebuEnSAUvrWVBGTxA7W+ZMNn5uyLlbOH7Nrs0bYOv6AOxQPqAo2UB0Z7vqlugjn+BUl0drDcWejBfDiPEC6jQA=="
        },
        "delayedMessagesRead": 354560
      },
      "signature": null
    }
  ]
}
```

위와 같은 JSON Format의 메시지 형태는 [Arbitrum Nitro 엔진](https://github.com/OffchainLabs/nitro)의 Go 언어 구현체에서 marshalling 된 것으로 `BroadcastMessage` 및 `BroadCastFeedMessage` 라는 `struct` 타입으로 구현되어있는데, 아래와 같은 [코드](https://github.com/OffchainLabs/nitro/blob/4941f52a1eab6002feff07146f0937dfd4ce53d6/broadcaster/message/message.go#L29)로 확인해 볼 수 있다. Nitro는 Geth를 기초로하여 만든 Arbitrum의 L2 구현용 엔진이다.

```go
type BroadcastMessage struct {
	Version int `json:"version"`
	// TODO better name than messages since there are different types of messages
	Messages                       []*BroadcastFeedMessage         `json:"messages,omitempty"`
	ConfirmedSequenceNumberMessage *ConfirmedSequenceNumberMessage `json:"confirmedSequenceNumberMessage,omitempty"`
}

type BroadcastFeedMessage struct {
	SequenceNumber arbutil.MessageIndex           `json:"sequenceNumber"`
	Message        arbostypes.MessageWithMetadata `json:"message"`
	BlockHash      *common.Hash                   `json:"blockHash,omitempty"`
	Signature      []byte                         `json:"signature"`

	CumulativeSumMsgSize uint64 `json:"-"`
}

type ConfirmedSequenceNumberMessage struct {
	SequenceNumber arbutil.MessageIndex `json:"sequenceNumber"`
}
```

또, `BroadcastFeedMessage` 구조체에 하위에 정의된 `Message` 는 `arbostypes.MessageWithMetadata` 타입으로 지정되어 있는데, 해당 타입은 Nitro의 `arbos` 패키지 하위에 아래와 같은 [코드](https://github.com/OffchainLabs/nitro/blob/4941f52a1eab6002feff07146f0937dfd4ce53d6/arbos/arbostypes/messagewithmeta.go#L17)로 구현되어 있다.

```go
type MessageWithMetadata struct {
	Message             *L1IncomingMessage `json:"message"`
	DelayedMessagesRead uint64             `json:"delayedMessagesRead"`
}

type MessageWithMetadataAndBlockHash struct {
	MessageWithMeta MessageWithMetadata
	BlockHash       *common.Hash
}
```

이와 별개로, `MessageWithMetadata` 구조체 하위에 정의된 `Message` 는 `*L1IncomingMessage` 타입으로 지정되어있고, 해당 구현체는 아래의 [코드](https://github.com/OffchainLabs/nitro/blob/4941f52a1eab6002feff07146f0937dfd4ce53d6/arbos/arbostypes/incomingmessage.go#L63)와 같이 구현되어 있다.

```go
type L1IncomingMessage struct {
	Header *L1IncomingMessageHeader `json:"header"`
	L2msg  []byte                   `json:"l2Msg"`

	// Only used for `L1MessageType_BatchPostingReport`
	BatchGasCost *uint64 `json:"batchGasCost,omitempty" rlp:"optional"`
}

type L1IncomingMessageHeader struct {
	Kind        uint8          `json:"kind"`
	Poster      common.Address `json:"sender"`
	BlockNumber uint64         `json:"blockNumber"`
	Timestamp   uint64         `json:"timestamp"`
	RequestId   *common.Hash   `json:"requestId" rlp:"nilList"`
	L1BaseFee   *big.Int       `json:"baseFeeL1"`
}
```

위 코드에서 `L2msg` 필드가 `[]byte` 타입으로 정의되어 있는 것을 알 수 있고, 앞선 JSON WebSocket Response의 에제에서는 `BAL40oKksUiElQL5AISg7rsAgxb6o5SZbYNoIF2DT...(중략)...ejBfDiPEC6jQA==` 값이었다.

해당 값은 base64 인코딩된 형태인데, 해당 `L2msg` 필드를 base64 디코딩하고 적절한 파싱 과정을 거치면 우리가 원하는 트랜잭션들의 정보를 얻을 수 있음을 추측할 수 있다. 이러한 메시지를 트랜잭션으로 디코딩 및 파싱하는 일련의 과정은 Nitro의 `arbos` 패키지 내에서 `ParseL2Transactions()` 라는 함수를 통해 이루어지게되고, 해당 [코드](https://github.com/OffchainLabs/nitro/blob/4941f52a1eab6002feff07146f0937dfd4ce53d6/arbos/parse_l2.go#L23)는 아래와 같다.

```go,linenos,hl_lines=8
func ParseL2Transactions(msg *arbostypes.L1IncomingMessage, chainId *big.Int) (types.Transactions, error) {
	if len(msg.L2msg) > arbostypes.MaxL2MessageSize {
		// ignore the message if l2msg is too large
		return nil, errors.New("message too large")
	}
	switch msg.Header.Kind {
	case arbostypes.L1MessageType_L2Message:
		return parseL2Message(bytes.NewReader(msg.L2msg), msg.Header.Poster, msg.Header.Timestamp, msg.Header.RequestId, chainId, 0)
	case arbostypes.L1MessageType_Initialize:
		return nil, errors.New("ParseL2Transactions encounted initialize message (should've been handled explicitly at genesis)")
	case arbostypes.L1MessageType_EndOfBlock:
		return nil, nil
	case arbostypes.L1MessageType_L2FundedByL1:
		if len(msg.L2msg) < 1 {
			return nil, errors.New("L2FundedByL1 message has no data")
		}
		if msg.Header.RequestId == nil {
			return nil, errors.New("cannot issue L2 funded by L1 tx without L1 request id")
		}
		kind := msg.L2msg[0]
		depositRequestId := crypto.Keccak256Hash(msg.Header.RequestId[:], arbmath.U256Bytes(common.Big0))
		unsignedRequestId := crypto.Keccak256Hash(msg.Header.RequestId[:], arbmath.U256Bytes(common.Big1))
		tx, err := parseUnsignedTx(bytes.NewReader(msg.L2msg[1:]), msg.Header.Poster, &unsignedRequestId, chainId, kind)
		if err != nil {
			return nil, err
		}
		deposit := types.NewTx(&types.ArbitrumDepositTx{
			ChainId:     chainId,
			L1RequestId: depositRequestId,
			// Matches the From of parseUnsignedTx
			To:    msg.Header.Poster,
			Value: tx.Value(),
		})
		return types.Transactions{deposit, tx}, nil
	case arbostypes.L1MessageType_SubmitRetryable:
		tx, err := parseSubmitRetryableMessage(bytes.NewReader(msg.L2msg), msg.Header, chainId)
		if err != nil {
			return nil, err
		}
		return types.Transactions{tx}, nil
	case arbostypes.L1MessageType_BatchForGasEstimation:
		return nil, errors.New("L1 message type BatchForGasEstimation is unimplemented")
	case arbostypes.L1MessageType_EthDeposit:
		tx, err := parseEthDepositMessage(bytes.NewReader(msg.L2msg), msg.Header, chainId)
		if err != nil {
			return nil, err
		}
		return types.Transactions{tx}, nil
	case arbostypes.L1MessageType_RollupEvent:
		log.Debug("ignoring rollup event message")
		return types.Transactions{}, nil
	case arbostypes.L1MessageType_BatchPostingReport:
		tx, err := parseBatchPostingReportMessage(bytes.NewReader(msg.L2msg), chainId, msg.BatchGasCost)
		if err != nil {
			return nil, err
		}
		return types.Transactions{tx}, nil
	case arbostypes.L1MessageType_Invalid:
		// intentionally invalid message
		return nil, errors.New("invalid message")
	default:
		// invalid message, just ignore it
		return nil, fmt.Errorf("invalid message type %v", msg.Header.Kind)
	}
}
```

Arbitrum의 Transaction Type은 아래와 같이 기존 Geth에서 정의된 것들에 더해, Arbitrum Nitro에서 특별히 정의한 유형이 더 추가되었다. 당연한 얘기지만 각각의 Transaction Type마다 역할과 의미가 다르다. 그에 따라, 트랜잭션의 유형마다 `switch-case` 문을 통해 분기되어 적절한 과정을 거쳐 Transaction 파싱이 이루어져 반환된다. 특히, 우리가 다루는 L2 메세지의 파싱은 `ParseL2Transactions()` 함수의 8번째 줄 `parseL2Message()` 함수를 거친다.

```go
// Transaction types.
const (
	ArbitrumDepositTxType         = 0x64
	ArbitrumUnsignedTxType        = 0x65
	ArbitrumContractTxType        = 0x66
	ArbitrumRetryTxType           = 0x68
	ArbitrumSubmitRetryableTxType = 0x69
	ArbitrumInternalTxType        = 0x6A
	ArbitrumLegacyTxType          = 0x78

	LegacyTxType     = 0x00
	AccessListTxType = 0x01
	DynamicFeeTxType = 0x02
	BlobTxType       = 0x03
)
```

`ParseL2Transactions()` 함수를 더 자세히 살펴보면, 첫 번째 인자로 Arbitrum Sequencer Feed로 부터 수신한 메시지의 내부의 타입인 `*arbostypes.L1IncomingMessage` 자료형의 `msg` 변수와 Arbitrum의 chainId 인 `42161` 값을 전달하게 되면, 파싱 과정에 오류가 없는 경우 파싱된 트랜잭션을 Geth에서 정의된 타입인 `Transcations` 타입으로 정상적으로 변환 받을 수 있다.

```go
func ParseL2Transactions(msg *arbostypes.L1IncomingMessage, chainId *big.Int) (types.Transactions, error)
```

참고로, 변환된 `Transactions` 타입의 값 내부에 존재하는 각각의 `Transaction` 타입 트랜잭션들은 Geth에서 구현된 `MarshalJSON()` 메소드를 통해 Human Readable한 형태로 트랜잭션의 각 필드들을 확인 할 수 있다. 해당 [코드](https://github.com/OffchainLabs/go-ethereum/blob/8e3ba695c6b07592383528855652db25f93865c2/core/types/transaction_marshalling.go#L99)는 아래와 같고, Nitro에서는 Geth의 기존 Transaction Type에서 Arbitrum 에서 별도 정의한 Transaction Type까지 Marshalling이 되도록 잘 정의되었다.

```go
// MarshalJSON marshals as JSON with a hash.
func (tx *Transaction) MarshalJSON() ([]byte, error) {
	var enc txJSON
	// These are set for all tx types.
	enc.Hash = tx.Hash()
	enc.Type = hexutil.Uint64(tx.Type())

	// Arbitrum: set to 0 for compatibility
	var zero uint64
	enc.Nonce = (*hexutil.Uint64)(&zero)
	enc.Gas = (*hexutil.Uint64)(&zero)
	enc.GasPrice = (*hexutil.Big)(common.Big0)
	enc.Value = (*hexutil.Big)(common.Big0)
	enc.Input = (*hexutil.Bytes)(&[]byte{})
	enc.V = (*hexutil.Big)(common.Big0)
	enc.R = (*hexutil.Big)(common.Big0)
	enc.S = (*hexutil.Big)(common.Big0)

	// Other fields are set conditionally depending on tx type.
	switch itx := tx.inner.(type) {
	case *LegacyTx:
		enc.Nonce = (*hexutil.Uint64)(&itx.Nonce)
		enc.To = tx.To()
		enc.Gas = (*hexutil.Uint64)(&itx.Gas)
		enc.GasPrice = (*hexutil.Big)(itx.GasPrice)
		enc.Value = (*hexutil.Big)(itx.Value)
		enc.Input = (*hexutil.Bytes)(&itx.Data)
		enc.V = (*hexutil.Big)(itx.V)
		enc.R = (*hexutil.Big)(itx.R)
		enc.S = (*hexutil.Big)(itx.S)
		if tx.Protected() {
			enc.ChainID = (*hexutil.Big)(tx.ChainId())
		}

	case *AccessListTx:
		enc.ChainID = (*hexutil.Big)(itx.ChainID)
		enc.Nonce = (*hexutil.Uint64)(&itx.Nonce)
		enc.To = tx.To()
		enc.Gas = (*hexutil.Uint64)(&itx.Gas)
		enc.GasPrice = (*hexutil.Big)(itx.GasPrice)
		enc.Value = (*hexutil.Big)(itx.Value)
		enc.Input = (*hexutil.Bytes)(&itx.Data)
		enc.AccessList = &itx.AccessList
		enc.V = (*hexutil.Big)(itx.V)
		enc.R = (*hexutil.Big)(itx.R)
		enc.S = (*hexutil.Big)(itx.S)
		yparity := itx.V.Uint64()
		enc.YParity = (*hexutil.Uint64)(&yparity)

	case *DynamicFeeTx:
		enc.ChainID = (*hexutil.Big)(itx.ChainID)
		enc.Nonce = (*hexutil.Uint64)(&itx.Nonce)
		enc.To = tx.To()
		enc.Gas = (*hexutil.Uint64)(&itx.Gas)
		enc.MaxFeePerGas = (*hexutil.Big)(itx.GasFeeCap)
		enc.MaxPriorityFeePerGas = (*hexutil.Big)(itx.GasTipCap)
		enc.Value = (*hexutil.Big)(itx.Value)
		enc.Input = (*hexutil.Bytes)(&itx.Data)
		enc.AccessList = &itx.AccessList
		enc.V = (*hexutil.Big)(itx.V)
		enc.R = (*hexutil.Big)(itx.R)
		enc.S = (*hexutil.Big)(itx.S)
		yparity := itx.V.Uint64()
		enc.YParity = (*hexutil.Uint64)(&yparity)

	case *ArbitrumLegacyTxData:
		enc.Nonce = (*hexutil.Uint64)(&itx.Nonce)
		enc.Gas = (*hexutil.Uint64)(&itx.Gas)
		enc.GasPrice = (*hexutil.Big)(itx.GasPrice)
		enc.Value = (*hexutil.Big)(itx.Value)
		enc.Input = (*hexutil.Bytes)(&itx.Data)
		enc.To = tx.To()
		enc.V = (*hexutil.Big)(itx.V)
		enc.R = (*hexutil.Big)(itx.R)
		enc.S = (*hexutil.Big)(itx.S)
		enc.EffectiveGasPrice = (*hexutil.Uint64)(&itx.EffectiveGasPrice)
		enc.L1BlockNumber = (*hexutil.Uint64)(&itx.L1BlockNumber)
		enc.From = itx.Sender
	case *ArbitrumInternalTx:
		enc.ChainID = (*hexutil.Big)(itx.ChainId)
		enc.Input = (*hexutil.Bytes)(&itx.Data)
	case *ArbitrumDepositTx:
		enc.RequestId = &itx.L1RequestId
		enc.From = &itx.From
		enc.ChainID = (*hexutil.Big)(itx.ChainId)
		enc.Value = (*hexutil.Big)(itx.Value)
		enc.To = tx.To()
	case *ArbitrumUnsignedTx:
		enc.From = &itx.From
		enc.ChainID = (*hexutil.Big)(itx.ChainId)
		enc.Nonce = (*hexutil.Uint64)(&itx.Nonce)
		enc.Gas = (*hexutil.Uint64)(&itx.Gas)
		enc.MaxFeePerGas = (*hexutil.Big)(itx.GasFeeCap)
		enc.Value = (*hexutil.Big)(itx.Value)
		enc.Input = (*hexutil.Bytes)(&itx.Data)
		enc.To = tx.To()
	case *ArbitrumContractTx:
		enc.RequestId = &itx.RequestId
		enc.From = &itx.From
		enc.ChainID = (*hexutil.Big)(itx.ChainId)
		enc.Gas = (*hexutil.Uint64)(&itx.Gas)
		enc.MaxFeePerGas = (*hexutil.Big)(itx.GasFeeCap)
		enc.Value = (*hexutil.Big)(itx.Value)
		enc.Input = (*hexutil.Bytes)(&itx.Data)
		enc.To = tx.To()
	case *ArbitrumRetryTx:
		enc.From = &itx.From
		enc.TicketId = &itx.TicketId
		enc.RefundTo = &itx.RefundTo
		enc.ChainID = (*hexutil.Big)(itx.ChainId)
		enc.Nonce = (*hexutil.Uint64)(&itx.Nonce)
		enc.Gas = (*hexutil.Uint64)(&itx.Gas)
		enc.MaxFeePerGas = (*hexutil.Big)(itx.GasFeeCap)
		enc.Value = (*hexutil.Big)(itx.Value)
		enc.Input = (*hexutil.Bytes)(&itx.Data)
		enc.MaxRefund = (*hexutil.Big)(itx.MaxRefund)
		enc.SubmissionFeeRefund = (*hexutil.Big)(itx.SubmissionFeeRefund)
		enc.To = tx.To()
	case *ArbitrumSubmitRetryableTx:
		enc.RequestId = &itx.RequestId
		enc.From = &itx.From
		enc.L1BaseFee = (*hexutil.Big)(itx.L1BaseFee)
		enc.DepositValue = (*hexutil.Big)(itx.DepositValue)
		enc.Beneficiary = &itx.Beneficiary
		enc.RefundTo = &itx.FeeRefundAddr
		enc.MaxSubmissionFee = (*hexutil.Big)(itx.MaxSubmissionFee)
		enc.ChainID = (*hexutil.Big)(itx.ChainId)
		enc.Gas = (*hexutil.Uint64)(&itx.Gas)
		enc.MaxFeePerGas = (*hexutil.Big)(itx.GasFeeCap)
		enc.RetryTo = itx.RetryTo
		enc.RetryValue = (*hexutil.Big)(itx.RetryValue)
		enc.RetryData = (*hexutil.Bytes)(&itx.RetryData)
		data := itx.data()
		enc.Input = (*hexutil.Bytes)(&data)
		enc.To = tx.To()

	case *BlobTx:
		enc.ChainID = (*hexutil.Big)(itx.ChainID.ToBig())
		enc.Nonce = (*hexutil.Uint64)(&itx.Nonce)
		enc.Gas = (*hexutil.Uint64)(&itx.Gas)
		enc.MaxFeePerGas = (*hexutil.Big)(itx.GasFeeCap.ToBig())
		enc.MaxPriorityFeePerGas = (*hexutil.Big)(itx.GasTipCap.ToBig())
		enc.MaxFeePerBlobGas = (*hexutil.Big)(itx.BlobFeeCap.ToBig())
		enc.Value = (*hexutil.Big)(itx.Value.ToBig())
		enc.Input = (*hexutil.Bytes)(&itx.Data)
		enc.AccessList = &itx.AccessList
		enc.BlobVersionedHashes = itx.BlobHashes
		enc.To = tx.To()
		enc.V = (*hexutil.Big)(itx.V.ToBig())
		enc.R = (*hexutil.Big)(itx.R.ToBig())
		enc.S = (*hexutil.Big)(itx.S.ToBig())
		yparity := itx.V.Uint64()
		enc.YParity = (*hexutil.Uint64)(&yparity)
		if sidecar := itx.Sidecar; sidecar != nil {
			enc.Blobs = itx.Sidecar.Blobs
			enc.Commitments = itx.Sidecar.Commitments
			enc.Proofs = itx.Sidecar.Proofs
		}
	}
	return json.Marshal(&enc)
}
```

이렇게 Arbitrum Sequencer Feed 메시지로부터 파싱된 트랜잭션들이, 실제로 Arbitrum One Network에 같은 순서로 조작없이 그대로 반영되었는지 각각의 트랜잭션의 Hash 값으로도 확인할 수 있다.

한 단위의 트랜잭션에 대해 Transaction Hash값을 통해 일일히 검증할 수도 있지만, 노드에 너무 빈번한(redundant) 요청을 발생시킬 수 있으므로, 단일 트랜잭션 단위로 검증하기보다 블록 단위의 검증을 하는 것이 더 효율적인 방법일 수 있다.

블록 단위의 메시지 검증을 위해서, EVM에서는 각각의 블록이 `Transactions Root` 라는 정보를 가지고 있는데, 이는 블록 내부의 트랜잭션들의 Merkle Trie 연산을 통해 도출된 Root Hash 값이다. 이 값을 통해, 우리가 수신한 메시지의 `Transactions Root` 값과 실제로 Arbitrum One Network에서 처리된 `Transactions Root` 값을 비교함으로써 검증할 수 있다.

`Transaction Root`를 계산하는 Geth의 로직은 다음과 같다. 먼저, Geth에서 정의한 Block의 Header 구조체의 정의에서, 아래와 같은 [코드](https://github.com/OffchainLabs/go-ethereum/blob/8e3ba695c6b07592383528855652db25f93865c2/core/types/block.go#L80)로 JSON Marshalling이 되었을 때의 `transactionsRoot` 필드 값을 Geth 내부적으로는 `TxHash` 라는 필드로 처리하고 있다.

```go,hl_lines=7
// Header represents a block header in the Ethereum blockchain.
type Header struct {
	ParentHash  common.Hash    `json:"parentHash"       gencodec:"required"`
	UncleHash   common.Hash    `json:"sha3Uncles"       gencodec:"required"`
	Coinbase    common.Address `json:"miner"`
	Root        common.Hash    `json:"stateRoot"        gencodec:"required"`
	TxHash      common.Hash    `json:"transactionsRoot" gencodec:"required"`
	ReceiptHash common.Hash    `json:"receiptsRoot"     gencodec:"required"`
	Bloom       Bloom          `json:"logsBloom"        gencodec:"required"`
	Difficulty  *big.Int       `json:"difficulty"       gencodec:"required"`
	Number      *big.Int       `json:"number"           gencodec:"required"`
	GasLimit    uint64         `json:"gasLimit"         gencodec:"required"`
	GasUsed     uint64         `json:"gasUsed"          gencodec:"required"`
	Time        uint64         `json:"timestamp"        gencodec:"required"`
	Extra       []byte         `json:"extraData"        gencodec:"required"`
	MixDigest   common.Hash    `json:"mixHash"`
	Nonce       BlockNonce     `json:"nonce"`

	// BaseFee was added by EIP-1559 and is ignored in legacy headers.
	BaseFee *big.Int `json:"baseFeePerGas" rlp:"optional"`

	// WithdrawalsHash was added by EIP-4895 and is ignored in legacy headers.
	WithdrawalsHash *common.Hash `json:"withdrawalsRoot" rlp:"optional"`

	// BlobGasUsed was added by EIP-4844 and is ignored in legacy headers.
	BlobGasUsed *uint64 `json:"blobGasUsed" rlp:"optional"`

	// ExcessBlobGas was added by EIP-4844 and is ignored in legacy headers.
	ExcessBlobGas *uint64 `json:"excessBlobGas" rlp:"optional"`

	// ParentBeaconRoot was added by EIP-4788 and is ignored in legacy headers.
	ParentBeaconRoot *common.Hash `json:"parentBeaconBlockRoot" rlp:"optional"`

	// RequestsHash was added by EIP-7685 and is ignored in legacy headers.
	RequestsHash *common.Hash `json:"requestsHash" rlp:"optional"`
}
```

그리고, Geth 에서는 각각의 Block이 생성될 때 `NewBlock()` 이라는 함수가 실행되는데, [코드](https://github.com/OffchainLabs/go-ethereum/blob/8e3ba695c6b07592383528855652db25f93865c2/core/types/block.go#L257)는 아래와 같다. 특히, `b.header.TxHash = DeriveSha(Transactions(txs), hasher)` 와 같이, `TransactionsRoot` 가 `DeriveSha()` 함수를 통해 블록에 담길 트랜잭션의 정보와, `TrieHasher` 라는 타입의 `hasher` 변수를 통해 이루어지는 것을 알 수 있다.

```go,hl_lines=15
func NewBlock(header *Header, body *Body, receipts []*Receipt, hasher TrieHasher) *Block {
	if body == nil {
		body = &Body{}
	}
	var (
		b           = NewBlockWithHeader(header)
		txs         = body.Transactions
		uncles      = body.Uncles
		withdrawals = body.Withdrawals
	)

	if len(txs) == 0 {
		b.header.TxHash = EmptyTxsHash
	} else {
		b.header.TxHash = DeriveSha(Transactions(txs), hasher)
		b.transactions = make(Transactions, len(txs))
		copy(b.transactions, txs)
	}

	if len(receipts) == 0 {
		b.header.ReceiptHash = EmptyReceiptsHash
	} else {
		b.header.ReceiptHash = DeriveSha(Receipts(receipts), hasher)
		// Receipts must go through MakeReceipt to calculate the receipt's bloom
		// already. Merge the receipt's bloom together instead of recalculating
		// everything.
		b.header.Bloom = MergeBloom(receipts)
	}

	if len(uncles) == 0 {
		b.header.UncleHash = EmptyUncleHash
	} else {
		b.header.UncleHash = CalcUncleHash(uncles)
		b.uncles = make([]*Header, len(uncles))
		for i := range uncles {
			b.uncles[i] = CopyHeader(uncles[i])
		}
	}

	if withdrawals == nil {
		b.header.WithdrawalsHash = nil
	} else if len(withdrawals) == 0 {
		b.header.WithdrawalsHash = &EmptyWithdrawalsHash
		b.withdrawals = Withdrawals{}
	} else {
		hash := DeriveSha(Withdrawals(withdrawals), hasher)
		b.header.WithdrawalsHash = &hash
		b.withdrawals = slices.Clone(withdrawals)
	}

	return b
}
```

더 깊이 나아가서 `DeriveSha()` 함수의 정의를 살펴볼 수도 있겠지만, 해당 아티클에서는 Arbitrum Sequencer Feed를 통한 검증을 다루고자 하므로 세부 로직은 생략한다. `DeriveSha()` 함수의 첫 번째 인자로는 트랜잭션들이 전달되고, 두 번째 인자로는 `trie.NewStackTrie(nil)` 가 전달되어야 트랜잭션의 Merkle Trie Root인,`Transactions Root`가 계산된다는 것만 알아두자.

그런데, 실제 Arbitrum의 Block의 생성이 될 때에는, 아래와 같은 [코드](https://github.com/OffchainLabs/nitro/blob/master/arbos/block_processor.go#L200)로, 기존 처리될 트랜잭션들의 가장 맨 앞에 `Start Block` 이라는 트랜잭션이 삽입되게 된다.

```go
// Prepend a tx before all others to touch up the state (update the L1 block num, pricing pools, etc)
startTx := InternalTxStartBlock(chainConfig.ChainID, l1Header.L1BaseFee, l1BlockNum, header, lastBlockHeader)
txes = append(types.Transactions{types.NewTx(startTx)}, txes...)
```

이는 Arbiscan의 각각의 블록에 포함된 트랜잭션들을 보면 바로 알 수 있는데, 매 블록마다 가장 앞 트랜잭션은 `Start Block`이라는 트랜잭션이 끼어있게 된다. 임의로 아무 블록 높이 값을 지정해서 확인해봐도 첫 번째로 포함된 트랜잭션은 항상 `Start Block` 트랜잭션이다.

{{ figure(src="./img/arbitrumSeqFeed.png", alt="arbitrumSeqFeed", caption="블록 높이 268544083의 첫 번째 트랜잭션도 'Start Block'") }}

`Start Block` 트랜잭션은, `Transactions Root` 를 계산하는 데에 있어서 당연히 필요한 트랜잭션이지만, Arbitrum Sequencer Feed 에서는 `Start Block` 트랜잭션이 포함되지 않은 상태로 송신되고 있다. 이에, `Start Block` 트랜잭션은 Sequencer Feed 발행 후에 이루어지는 블록 생성 바로 직전에 삽입되는 것이므로, 우리도 마찬가지로 내부적으로 `Start Block`을 삽입해야 한다.

`Start Block` 트랜잭션이 생성되는 Nitro의 구현은 `InternalTxStartBlock()` 함수 등으로 이루어져 있고, [코드](https://github.com/OffchainLabs/nitro/blob/4941f52a1eab6002feff07146f0937dfd4ce53d6/arbos/internal_tx.go#L24)는 아래 같다.

```go,hl_lines=15
func InternalTxStartBlock(
	chainId,
	l1BaseFee *big.Int,
	l1BlockNum uint64,
	header,
	lastHeader *types.Header,
) *types.ArbitrumInternalTx {

	l2BlockNum := header.Number.Uint64()
	timePassed := header.Time - lastHeader.Time

	if l1BaseFee == nil {
		l1BaseFee = big.NewInt(0)
	}
	data, err := util.PackInternalTxDataStartBlock(l1BaseFee, l1BlockNum, l2BlockNum, timePassed)
	if err != nil {
		panic(fmt.Sprintf("Failed to pack internal tx %v", err))
	}
	return &types.ArbitrumInternalTx{
		ChainId: chainId,
		Data:    data,
	}
}
```

코드 중 특히, `util.PackInternalTxDataStartBlock(l1BaseFee, l1BlockNum, l2BlockNum, timePassed)` 와 같은 방법으로 `Start Block` 트랜잭션의 `Data` 필드를 연산하고 있는데, `l1BaseFee` 및 `l1BlockNum`, `l2BlockNum`, `timePassed`와 같은 인자들을 받고 있다. 아래는 해당 인자들의 계산 방법이다. (앞선 Sequencer Feed Message의 예제로 보여준 단위 Response를 아래에서 그대로 다시 작성하였다)

- `l1BaseFee` : Sequencer Feed 에서 `baseFeeL1` 필드의 값으로 얻을 수 있음
- `l1BlockNum` : Sequencer Feed 에서 `blockNumber` 필드의 값으로 얻을 수 있음
- `l2BlockNum` : Sequencer Feed 에서 `sequenceNumber` 필드의 값에 `22207817` (Arbitrum One Genesis Block Number) 를 더한 값으로 얻을 수 있음 (이는 앞서 언급한 Arbitrum Official document의 [How to read sequencer feed](https://docs.arbitrum.io/run-arbitrum-node/sequencer/read-sequencer-feed)에 설명되어 있다)
- `timePassed` : Sequencer Feed 에서 해당 메시지 `timestamp` 필드의 값에서 해당 메시지 바로 직전에 수신한 메시지의 `timestamp` 필드의 값을 뺀 값으로 얻을 수 있음 (즉, 메시지 검증을 위해서는 검증하고자 할 메시지의 바로 직전 메시지의 `timestamp` 값이 필요하다)

```json
{
  "version": 1,
  "messages": [
    {
      "sequenceNumber": 25757171,
      "message": {
        "message": {
          "header": {
            "kind": 3,
            "sender": "0xa4b000000000000000000073657175656e636572",
            "blockNumber": 16238523,
            "timestamp": 1671691403,
            "requestId": null,
            "baseFeeL1": null
          },
          "l2Msg": "BAL40oKksUiElQL5AISg7rsAgxb6o5SZbYNoIF2DTixsqDpD2xII9GJLG4C4ZAhh6N0AAAAAAAAAAAAAAAC7EQiq1R1VYgL3/oXgvD921hYRyAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAArAAaAkebuEnSAUvrWVBGTxA7W+ZMNn5uyLlbOH7Nrs0bYOv6AOxQPqAo2UB0Z7vqlugjn+BUl0drDcWejBfDiPEC6jQA=="
        },
        "delayedMessagesRead": 354560
      },
      "signature": null
    }
  ]
}
```

이를 통해, `Start Block` 트랜잭션을 우리가 연산 및 빌드하여, 시퀀서 메시지로부터 Decoding 및 Parsing한 Transaction의 맨 앞에 삽입하여 `Transactions Root`를 구한 후, 해당 메시지 및 직전 메시지의 `timestamp` 값으로부터 연산 된 `l2BlockNum` 을 `viem` 등의 라이브러리를 이용한 RPC 호출 등을 통해 실제 Arbitrum One에서 처리된 Block의 `Transactions Root` 값과 같은지 비교하여, Arbitrum Sequencer가 정상적으로 동작하고 있는지 Validation 할 수 있다.

## WASM Compile

이렇게 Go 언어로 작성된 트랜잭션 파싱 과정을 웹 어셈블리 언어로 컴파일하면 웹에서도 사용할 수 있다. 내가 직접 작성한 [arbfeedtowasm](https://github.com/gloomydumber/arbfeedtowasm) 이라는 레포지토리에서 작성한 아래와 같은 Go 언어 코드를 컴파일하면 Arbitrum Sequencer Feed 메시지를 파싱하는 `.wasm` 파일을 생성할 수 있다. 파싱에 사용된 함수는 모두 Nitro를 git submodule로 설정하고 임포트하여 과정을 진행하였다.

```go
package jsutil

import (
	"encoding/json"
	"syscall/js"

	"arbfeedtowasm/feedtypes"
	"arbfeedtowasm/operation"
	"arbfeedtowasm/utils"

	"github.com/ethereum/go-ethereum/common"
	"github.com/ethereum/go-ethereum/core/types"
)

func CalculateTransactionsRootWithStartTx(this js.Value, p []js.Value) interface{} {
	var sequencerMessage = p[0].String()
	var lastTimestamp = uint64(p[1].Int())

	var msg feedtypes.IncomingMessage = utils.ParseIncomingMessage(sequencerMessage)
	var txns types.Transactions = operation.ParseL2TransactionsWithStartTx(msg, lastTimestamp)
	var txnsRoot common.Hash = operation.CalculateTransactionsRoot(txns)

	return js.ValueOf(txnsRoot.String())
}

func GetTransactionDetailsWithRoot(this js.Value, p []js.Value) interface{} {
	var sequencerMessage = p[0].String()
	var lastTimestamp = uint64(p[1].Int())

	var msg feedtypes.IncomingMessage = utils.ParseIncomingMessage(sequencerMessage)
	var txns types.Transactions = operation.ParseL2TransactionsWithStartTx(msg, lastTimestamp)
	var txnsRoot common.Hash = operation.CalculateTransactionsRoot(txns)

	response := feedtypes.Response{
		Result: "success",
		Data: &feedtypes.ResponseData{
			Transactions:     txns,
			TransactionsRoot: txnsRoot.String(),
		},
	}

	jsonResult, err := json.Marshal(response)
	if err != nil {
		response = feedtypes.Response{
			Result: "error",
			Error: &feedtypes.ErrorResponse{
				Message: "Failed to marshal response to JSON",
			},
		}
		jsonResult, _ = json.Marshal(response)
		return js.ValueOf(string(jsonResult))
	}

	return js.ValueOf(string(jsonResult))
}
```

Go 언어에서 웹 어셈블리로의 컴파일은 아래와 같은 명령어로 진행하면 된다.

```bash
$ GOJS=js GOARCH=wasm go build -o {resultFileName}.wasm {originFileName}.go
```

컴파일 된 `.wasm` 파일을 이용하면 웹 환경에서 WebSocket으로 Arbitrum Sequencer Feed를 수신한 결과를 파싱하고, 또 그 파싱한 결과와 Arbitrum RPC Node에 요청한 결과를 비교하여 검증할 수 있다.

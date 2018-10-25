# 기본 프로토타입

[![](../.gitbook/assets/image.png)](https://jeiwan.cc/posts/building-blockchain-in-go-part-1/)

## 소개

'블록체인'이란 21세기의 가장 혁신적인 기술 중 하나입니다. 이는 아직 발전하는 단계에 있으며, 이에 대한 잠재력은 완전히 실현되지 않았습니다. 본질적으로, 블록체인은 단지 분산된 레코드들의 데이터베이스라고 할 수 있습니다. 하지만 이는 비공개 데이터베이스가 아닌, 공개 데이터베이스라는 독특한 특징을 가지고 있습니다. 즉, 블록체인을 사용하는 모든 사용자들은 전체적으로 또는 부분적으로라도 이를 가지고 있을 수 있다는 것입니다. 그리고 새로운 레코드는 해당 데이터베이스의 다른 관리자들의 동의하에만 추가될 수 있습니다. 또한, 블록체인은 암호화폐와 스마트 계약을 가능하게 한다는 특징을 가지고 있습니다.

이 일련의 문서를 통해, 저희는 간단한 블록체인 구현을 기반으로 하는 단순화된 암호화폐를 제작할 것입니다.

## 블록

먼저, "블록체인"의 "블록"부분 부터 살펴보겠습니다. 블록체인에서 중요한 정보를 저장하는 것이 블록에 해당합니다. 예를 들어, 비트코인의 블록들은 암호화폐의 필수 요소인 트랜잭션을 저장하고 있습니다. 이 외에도 하나의 블록은 해당 블록의 버전, 그리고 현재 타임스탬프 및 이전 블록의 해시 값과 같은 일부 기술적인 정보들을 포함하고 있습니다.

해당 문서에서, 저희는 블록체인이나 비트코인 설명서에 설명되어 있는 방식 그대로 블록을 구현하지 않고, 오로지 중요한 정보만을 포함하는 단순화된 버전을 사용할 것입니다. 이에 구조는 다음과 같습니다.

```go
type Block struct {
    Timestamp     int64
    Data          []byte
    PrevBlockHash []byte
    Hash          []byte
}
```

`Timestamp`는 현재의 타임스탬프\(블록이 생성되었을 때\), `Data`는 블록에 포함되어 있는 실제 중요한 정보, `PrevBlockHash`는 이전 블록의 해시 값, 그리고 `Hash`는 해당 블록의 해시 값입니다. 비트코인 설명서에서, `TimeStamp`, `PrevBlockHash`, `Hash`는 독립된 데이터 구조를 형성하는 블록 헤더이고 트랙젝션\(저희의 경우 `Data`\)은 하나의 독립된 데이터 구조입니다. 여기서 저희는 단순하게 이들을 합쳤습니다.

그렇다면 저희는 이 해시 값들을 어떻게 계산해야 할까요? 해시 값을 계산하는 방법은 블록체인에서 가장 중요한 특징이며, 이는 블록체인을 안전하게 만드는 핵심기능입니다. 이 해시 값을 계산하는 것은 빠른 컴퓨터에서조차도 다소 시간이 걸릴 정도로 계산적으로 어려운 작업입니다. \(이것이 바로 사람들이 비트코인을 채굴하기 위해 강력한 GPU를 구매하는 이유입니다.\) 이것은 새로운 블록을 추가하는 것을 어렵게하여 추가된 후에도 수정이 불가능하도록 의도적으로 설계된 디자인입니다. 우리는 향후 문서에서 해당 메커니즘을 논의하고 구현할 예정입니다.

일단, 우리는 블록을 구성하는 필드들을 합친 후, 합쳐진 문자열들에 대한 SHA-256 해시 값을 계산할 것입니다. `SetHash` 함수는 다음과 같습니다:

```go
func (b *Block) SetHash() {
    timestamp := []byte(strconv.FormatInt(b.Timestamp, 10))
    headers := bytes.Join([][]byte{b.PrevBlockHash, b.Data, timestamp}, []byte{})
    hash := sha256.Sum256(headers)
​
    b.Hash = hash[:]
}
```

다음, Go 언어에 따라, 우리는 하나의 블록을 단순하게 생성하는 함수를 다음과 같이 구현할 것입니다:

```go
func NewBlock(data string, prevBlockHash []byte) *Block {
    block := &Block{time.Now().Unix(), []byte(data), prevBlockHash, []byte{}}
    block.SetHash()
    return block
}
```

이제, 블록을 위한 작업은 끝났습니다!

## 블록체인

지금부터는 블록체인을 구현해 봅시다. 본질적으로, 블록체인은 특정한 구조를 가지는 하나의 데이터베이스일 뿐입니다. 이 특정한 구조는 정렬되어 있고 거꾸로 된 링크드 리스트를 의미합니다. 블록들은 삽입된 순서대로 정렬되며, 각각의 블록들은 이전 블록과 연결되는 구조입니다. 이러한 구조로 인해서, 체인에서 가장 최신의 블록을 빠르게 얻을 수 있고, 해시 값으로부터 하나의 블록을 \(효율적으로\) 얻어낼 수 있습니다.

Go 언어에서 해당 구조는 배열과 맵을 사용하여 구현될 수 있습니다. 배열은 정렬된 해시 값들을 담을 수 있고\(배열은 Go 언어에서 정렬되어 있습니다.\), 맵은 `hash → block`쌍을 담을 수 있습니다. 하지만 저희의 블록체인 프로타입에서는 당장 해시 값들로부터 블록을 얻어낼 필요가 없기 때문에, 다음과 같이 단 하나의 배열만 사용할 것입니다.

```go
type Blockchain struct {
    blocks []*Block
}
```

이것이 저희의 첫 번째 블록체인입니다! 저는 이렇게 쉬울 줄 몰랐네요.😉

이제 블록을 추가하는 것이 가능하도록 해봅시다:

```go
func (bc *Blockchain) AddBlock(data string) {
    prevBlock := bc.blocks[len(bc.blocks)-1]
    newBlock := NewBlock(data, prevBlock.Hash)
    bc.blocks = append(bc.blocks, newBlock)
}
```

다 되었습니다! 이제 끝일까요?..

새로운 블록을 추가하기 위해서는 기존의 블록들이 필요하지만, 저희의 블록체인에서는 어떠한 블록도 존재하지 않습니다! 따라서, 모든 블록체인에서는 적어도 하나의 블록이 존재해야하고, 그 체인에서의 첫번째 블록은 **제네시스 블록**이라고 부릅니다. 그러면 이러한 하나의 블록을 생성하기 위한 함수를 다음과 같이 구현해 봅시다:

```go
func NewGenesisBlock() *Block {
    return NewBlock("Genesis Block", []byte{})
}
```

이제, 우리는 제네시스 블록을 가진 블록체인을 생성하는 함수를 다음과 같이 구현할 수 있습니다:

```go
func NewBlockchain() *Blockchain {
    return &Blockchain{[]*Block{NewGenesisBlock()}}
}
```

블록체인이 제대로 동작하는지 다음과 같이 확인해 봅시다:

```go
func main() {
    bc := NewBlockchain()
​
    bc.AddBlock("Send 1 BTC to Ivan")
    bc.AddBlock("Send 2 more BTC to Ivan")
​
    for _, block := range bc.blocks {
        fmt.Printf("Prev. hash: %x\n", block.PrevBlockHash)
        fmt.Printf("Data: %s\n", block.Data)
        fmt.Printf("Hash: %x\n", block.Hash)
        fmt.Println()
    }
}
```

출력 결과:

```go
Prev. hash:
Data: Genesis Block
Hash: aff955a50dc6cd2abfe81b8849eab15f99ed1dc333d38487024223b5fe0f1168
​
Prev. hash: aff955a50dc6cd2abfe81b8849eab15f99ed1dc333d38487024223b5fe0f1168
Data: Send 1 BTC to Ivan
Hash: d75ce22a840abb9b4e8fc3b60767c4ba3f46a0432d3ea15b71aef9fde6a314e1
​
Prev. hash: d75ce22a840abb9b4e8fc3b60767c4ba3f46a0432d3ea15b71aef9fde6a314e1
Data: Send 2 more BTC to Ivan
Hash: 561237522bb7fcfbccbc6fe0e98bbbde7427ffe01c6fb223f7562288ca2295d1
```

잘 작동합니다!

### 결론

저희는 가장 단순한 블록체인 프로토타입을 만들었습니다. 이는 이전의 블록과 연결된 각각의 블록들의 배열일 뿐입니다. 실제 블록체인은 훨씬 더 복잡합니다. 저희의 블록체인에서 새로운 블록들을 추가하는 것은 쉽고 빠르지만, 실제 블록체인에서 새로운 블록들을 추가하는 것은 몇 가지 작업이 더 필요합니다. 그 중 하나는 블록 추가를 위한 동의를 얻기 전에 무거운 계산을 수행해야하는 작업입니다.\(이 메커니즘은 작업증명이라고 부릅니다.\) 또한, 블록체인은 단일 의사 결정자가 없는 분산된 데이터베이스입니다. 따라서, 하나의 새로운 블록은 해당 네트워크의 다른 참여자들로부터 확인 및 승인을 받아야 합니다. \(이 메커니즘은 컨센서스라고 부릅니다.\) 그리고 아직 저희의 블록체인에서는 트랜잭션이 없습니다.

앞으로의 문서에서 우리는 이러한 기능들을 각각 살펴볼 것입니다.


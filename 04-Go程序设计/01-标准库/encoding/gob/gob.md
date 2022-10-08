[TOC]

### gob

#### What ?

package gob manages streams of gobs - binary values exchanged between an Encoder (transmitter) and a Decoder (receiver). A typical use is transporting arguments and results of remote procedure calls (RPCs) such as those provided by package "net/rpc".

`gob package` 是管理 `gob` 流的二进制值在编码器(发射器)和解码器(接收器)之间交换. 典型的用途是传输远程过程调用(rpc)的参数和结果, 如包"net/rpc"提供的参数和结果. 

#### How ?

##### Basic

~~~go
// Transmit
// Export Fields Or UnExport Fields
type P struct {
	X    int
	Y    int
	Z    int
	Name string
}

// Reviver
// U Can Pointer And Omit Field
type Q struct {
	X    *int32
	Y    *int32
	Name string
}

// This example shows the basic usage of the package:
// 1. Create an encoder,
// 2. transmit some values
// 3. receive them with a decoder.
func TestNormalGob(t *testing.T) {
	// Network
	var network bytes.Buffer

	// Initialize the encoder and decoder.
	// Write to Network
	encoder := gob.NewEncoder(&network)

	// encode
	encoder.Encode(P{X: 1, Y: 2, Z: 3, Name: "GodYao"})

	encoder.Encode(P{X: 4, Y: 5, Z: 6, Name: "Mike"})

	// ----------------------------------------------------------------

	// Normally enc and dec would be
	// bound to network connections and the encoder and decoder would
	// run in different processes.

	// ----------------------------------------------------------------

	// Initialize the decoder
	// Read From Network
	var q Q
	decoder := gob.NewDecoder(&network)

	// decode
	if err := decoder.Decode(&q); err != nil {
		t.Fatal(err)
	}
	t.Logf("%d, %d, %s", *q.X, *q.Y, q.Name)
	if err := decoder.Decode(&q); err != nil {
		t.Fatal(err)
	}
	t.Logf("%d, %d, %s", *q.X, *q.Y, q.Name)
}
~~~

说明 源与目标不用完全匹配对应; 目标可以缺失某些字段;源必须是导出字段;他们是根据名称来匹配的.

##### Transmit Interface 

~~~go
type LinkFace interface {
	GenerateLink() string
}

type Web struct {
	Proto  string
	Domain string
	Port   string
}

func (w Web) GenerateLink() string {
	return fmt.Sprintf("%s://%s:%s", w.Proto, w.Domain, w.Port)
}

// How to encode an interface value
func TestEmptyInterfaceGob(t *testing.T) {
	var network bytes.Buffer

	// We must register the concrete type for the encoder and decoder (which would
	// normally be on a separate machine from the encoder).
	// On each end, this tells the engine which concrete type is being
    // sent that implements the interface.
	// 必须为编码与解码注册具体的类型, 另一方面也是告知引擎正在发送那个具体类型来实现了接口
	gob.Register(&Web{})

	// Encode
	encoder := gob.NewEncoder(&network)

	var webSendFace LinkFace = &Web{
        Proto: "http", 
        Domain: "godyao.com.cn", 
        Port: "8090",
    }

	// Must Pointer to interface
	if err := encoder.Encode(&webSendFace); err != nil {
		t.Fatal(err)
	}

	// Decoder also Register gob.Register(&Web{})
	// Decode
	decoder := gob.NewDecoder(&network)
	var webReciverFace LinkFace
	if err := decoder.Decode(&webReciverFace); err != nil {
		t.Fatal(err)
	}
	t.Logf("%s", webReciverFace.GenerateLink())
}
~~~

##### Transmit Include  Interface{}

~~~go
type MessageSender struct {
	Name    string
	Seq     int64
	Request interface{}
}

type MessageReceiver struct {
	Name    string
	Seq     *int64
	Request interface{}
}

func TestGobEncodeAndDecodeIncludeMap(t *testing.T) {

	var network bytes.Buffer

	encoder := gob.NewEncoder(&network)

	// 如果值中含有interface类型, 必须先注册传输的具体类型 (如果该类型是 Map 或者 Struct)
	gob.Register(map[string]string{})

	request := map[string]string{
		"method": "gob",
		"mode":   "encode",
	}
	message := MessageSender{
		Name:    "Hello",
		Seq:     1001,
		Request: request,
	}

	if err := encoder.Encode(&message); err != nil {
		t.Fatal(err)
	}

	// Receiver
	dec := gob.NewDecoder(&network)
	var messageReceiver MessageReceiver
	dec.Decode(&messageReceiver)

	t.Log(messageReceiver)
}

func TestGobEncodeAndDecodeIncludeStruct(t *testing.T) {
	var network bytes.Buffer

	encoder := gob.NewEncoder(&network)

	// 如果值中含有interface类型, 必须先注册传输的具体类型 (如果该类型是 Map 或者 Struct)
	type Request struct {
		Method string
		Mode   string
	}

	gob.Register(Request{})

	request := &Request{
		Method: "gob",
		Mode:   "encode",
	}

	message := MessageSender{
		Name:    "Hello",
		Seq:     1001,
		Request: request,
	}

	if err := encoder.Encode(&message); err != nil {
		t.Fatal(err)
	}

	// Receiver
	dec := gob.NewDecoder(&network)
	var messageReceiver MessageReceiver
	dec.Decode(&messageReceiver)

	t.Log(messageReceiver)
}
~~~
```
go get -u github.com/pip-services4/pip-services4-go/pip-services4-grpc-go@latest
```

```
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative bin/protos/summator.proto 

```

```
package calculations

func Summator(x float32, y float32) float32 {
	z := x + y
	return z
}
```

```
import grpcctrl "github.com/pip-services4/pip-services4-go/pip-services4-grpc-go/controllers"
```

```
import (
    "mymodule/protos"
)

```

```
import (
    "mymodule/calculations"
)
```

```
import (
    "context"
)
```

```
// gRPC controller
type MyGrpcController struct {
	*grpcctrl.GrpcController
	protos.SummatorServer
}

func NewMyGrpcController() *MyGrpcController {
	c := &MyGrpcController{}
	c.GrpcController = grpcctrl.InheritGrpcController(c, "Summator")
	return c
}

func (c *MyGrpcController) Sum(ctx context.Context, req *protos.Number1) (result *protos.Number2, err error) {

	res := calculations.Summator(req.GetValue1(), req.GetValue2())
	result = &protos.Number2{Value: res}
	return result, nil
}

func (c *MyGrpcController) Register() {
	protos.RegisterSummatorServer(c.Endpoint.GetServer(), c)
}
```

```
import (
        cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

controller := NewMyGrpcController()
controller.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
	"connection.protocol", "http",
	"connection.host", "localhost",
	"connection.port", 50055,
))

controller.SetReferences(context.Background(), cref.NewEmptyReferences())
```

```
err := controller.Open(context.Background())
```

```
import (
    grpcclients "github.com/pip-services4/pip-services4-go/pip-services4-grpc-go/clients"
)
```

```
import (
    "mymodule/protos"
)
```

```
// gRPC client
type MyGrpcClient struct {
	*grpcclients.GrpcClient
}

func NewMyGrpcClient() *MyGrpcClient {
	dgc := MyGrpcClient{}
	dgc.GrpcClient = grpcclients.NewGrpcClient("Summator")
	return &dgc
}

func (c *MyGrpcClient) GetData(ctx context.Context, correlationId string, value1 float32, value2 float32) (result float32, err error) {

	req := &protos.Number1{Value1: value1, Value2: value2}
	reply := new(protos.Number2)
	err = c.Call("sum", correlationId, req, reply)
	if err != nil {
		return 0, err
	}
	result = reply.GetValue()

	return result, nil
}
```

```
import (
        cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
        cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
)

client := NewMyGrpcClient()
client.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
	"connection.protocol", "http",
	"connection.host", "localhost",
	"connection.port", 50055,
))

client.SetReferences(context.Background(), cref.NewEmptyReferences())
```

```
err := client.Open(context.Background())
```

```
result, err := client.GetData(context.Background(), "123", 3, 5) // Returns 8
```

```
import (
	"context"
	"mymodule/calculations"
	"mymodule/protos"
	"time"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	grpcctrl "github.com/pip-services4/pip-services4-go/pip-services4-grpc-go/controllers"
)

// gRPC controller
type MyGrpcController struct {
	*grpcctrl.GrpcController
	protos.SummatorServer
}

func NewMyGrpcController() *MyGrpcController {
	c := &MyGrpcController{}
	c.GrpcController = grpcctrl.InheritGrpcController(c, "Summator")
	return c
}

func (c *MyGrpcController) Sum(ctx context.Context, req *protos.Number1) (result *protos.Number2, err error) {

	res := calculations.Summator(req.GetValue1(), req.GetValue2())
	result = &protos.Number2{Value: res}
	return result, nil
}

func (c *MyGrpcController) Register() {
	protos.RegisterSummatorServer(c.Endpoint.GetServer(), c)
}

func main() {
	controller := NewMyGrpcController()
	controller.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
	"connection.protocol", "http",
	"connection.host", "localhost",
	"connection.port", 50055,
	))

	controller.SetReferences(context.Background(), cref.NewEmptyReferences())

	err := controller.Open(context.Background())

	if err != nil {
		panic(err)
	}

	<-time.After(1 * time.Hour)

	err = controller.Close(context.Background())

	if err != nil {
		panic(err)
	}
}
```

```
import (
	"context"
	"fmt"
	"mymodule/protos"

	cconf "github.com/pip-services4/pip-services4-go/pip-services4-components-go/config"
	cref "github.com/pip-services4/pip-services4-go/pip-services4-components-go/refer"
	grpcclients "github.com/pip-services4/pip-services4-go/pip-services4-grpc-go/clients"
)

// gRPC client
type MyGrpcClient struct {
	*grpcclients.GrpcClient
}

func NewMyGrpcClient() *MyGrpcClient {
	dgc := MyGrpcClient{}
	dgc.GrpcClient = grpcclients.NewGrpcClient("Summator")
	return &dgc
}

func (c *MyGrpcClient) GetData(ctx context.Context, correlationId string, value1 float32, value2 float32) (result float32, err error) {

	req := &protos.Number1{Value1: value1, Value2: value2}
	reply := new(protos.Number2)
	err = c.Call("sum", correlationId, req, reply)
	if err != nil {
		return 0, err
	}
	result = reply.GetValue()

	return result, nil
}

func main() {
	client := NewMyGrpcClient()
	client.Configure(context.Background(), cconf.NewConfigParamsFromTuples(
		"connection.protocol", "http",
		"connection.host", "localhost",
		"connection.port", 50055,
	))

	client.SetReferences(context.Background(), cref.NewEmptyReferences())

	err := client.Open(context.Background())

	// Function call and result
	result, err := client.GetData(context.Background(), "123", 3, 5) // Returns 8

	fmt.Print(result)

	err = client.Close(context.Background())

	fmt.Print(err)
}
```

```
syntax = "proto3";

option go_package = "./main";

message Number1 {
    float value1 = 1;
    float value2 = 2;
}

message Number2 {
    float value = 1;
}

service Summator {
    rpc sum(Number1) returns (Number2) {}
}

```

```
// Code generated by protoc-gen-go-grpc. DO NOT EDIT.

package protos

import (
	context "context"
	grpc "google.golang.org/grpc"
	codes "google.golang.org/grpc/codes"
	status "google.golang.org/grpc/status"
)

// This is a compile-time assertion to ensure that this generated file
// is compatible with the grpc package it is being compiled against.
// Requires gRPC-Go v1.32.0 or later.
const _ = grpc.SupportPackageIsVersion7

// SummatorClient is the client API for Summator service.
//
// For semantics around ctx use and closing/ending streaming RPCs, please refer to https://pkg.go.dev/google.golang.org/grpc/?tab=doc#ClientConn.NewStream.
type SummatorClient interface {
	Sum(ctx context.Context, in *Number1, opts ...grpc.CallOption) (*Number2, error)
}

type summatorClient struct {
	cc grpc.ClientConnInterface
}

func NewSummatorClient(cc grpc.ClientConnInterface) SummatorClient {
	return &summatorClient{cc}
}

func (c *summatorClient) Sum(ctx context.Context, in *Number1, opts ...grpc.CallOption) (*Number2, error) {
	out := new(Number2)
	err := c.cc.Invoke(ctx, "/Summator/sum", in, out, opts...)
	if err != nil {
		return nil, err
	}
	return out, nil
}

// SummatorServer is the server API for Summator service.
// All implementations must embed UnimplementedSummatorServer
// for forward compatibility
type SummatorServer interface {
	Sum(context.Context, *Number1) (*Number2, error)
	mustEmbedUnimplementedSummatorServer()
}

// UnimplementedSummatorServer must be embedded to have forward compatible implementations.
type UnimplementedSummatorServer struct {
}

func (UnimplementedSummatorServer) Sum(context.Context, *Number1) (*Number2, error) {
	return nil, status.Errorf(codes.Unimplemented, "method Sum not implemented")
}
func (UnimplementedSummatorServer) mustEmbedUnimplementedSummatorServer() {}

// UnsafeSummatorServer may be embedded to opt out of forward compatibility for this service.
// Use of this interface is not recommended, as added methods to SummatorServer will
// result in compilation errors.
type UnsafeSummatorServer interface {
	mustEmbedUnimplementedSummatorServer()
}

func RegisterSummatorServer(s grpc.ServiceRegistrar, srv SummatorServer) {
	s.RegisterService(&Summator_ServiceDesc, srv)
}

func _Summator_Sum_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
	in := new(Number1)
	if err := dec(in); err != nil {
		return nil, err
	}
	if interceptor == nil {
		return srv.(SummatorServer).Sum(ctx, in)
	}
	info := &grpc.UnaryServerInfo{
		Server:     srv,
		FullMethod: "/Summator/sum",
	}
	handler := func(ctx context.Context, req interface{}) (interface{}, error) {
		return srv.(SummatorServer).Sum(ctx, req.(*Number1))
	}
	return interceptor(ctx, in, info, handler)
}

// Summator_ServiceDesc is the grpc.ServiceDesc for Summator service.
// It's only intended for direct use with grpc.RegisterService,
// and not to be introspected or modified (even as a copy)
var Summator_ServiceDesc = grpc.ServiceDesc{
	ServiceName: "Summator",
	HandlerType: (*SummatorServer)(nil),
	Methods: []grpc.MethodDesc{
		{
			MethodName: "sum",
			Handler:    _Summator_Sum_Handler,
		},
	},
	Streams:  []grpc.StreamDesc{},
	Metadata: "bin/protos/summator.proto",
}


```

```
// Code generated by protoc-gen-go. DO NOT EDIT.
// versions:
// 	protoc-gen-go v1.27.1
// 	protoc        v3.17.3
// source: bin/protos/summator.proto

package protos

import (
	protoreflect "google.golang.org/protobuf/reflect/protoreflect"
	protoimpl "google.golang.org/protobuf/runtime/protoimpl"
	reflect "reflect"
	sync "sync"
)

const (
	// Verify that this generated code is sufficiently up-to-date.
	_ = protoimpl.EnforceVersion(20 - protoimpl.MinVersion)
	// Verify that runtime/protoimpl is sufficiently up-to-date.
	_ = protoimpl.EnforceVersion(protoimpl.MaxVersion - 20)
)

type Number1 struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Value1 float32 `protobuf:"fixed32,1,opt,name=value1,proto3" json:"value1,omitempty"`
	Value2 float32 `protobuf:"fixed32,2,opt,name=value2,proto3" json:"value2,omitempty"`
}

func (x *Number1) Reset() {
	*x = Number1{}
	if protoimpl.UnsafeEnabled {
		mi := &file_bin_protos_summator_proto_msgTypes[0]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *Number1) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*Number1) ProtoMessage() {}

func (x *Number1) ProtoReflect() protoreflect.Message {
	mi := &file_bin_protos_summator_proto_msgTypes[0]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use Number1.ProtoReflect.Descriptor instead.
func (*Number1) Descriptor() ([]byte, []int) {
	return file_bin_protos_summator_proto_rawDescGZIP(), []int{0}
}

func (x *Number1) GetValue1() float32 {
	if x != nil {
		return x.Value1
	}
	return 0
}

func (x *Number1) GetValue2() float32 {
	if x != nil {
		return x.Value2
	}
	return 0
}

type Number2 struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Value float32 `protobuf:"fixed32,1,opt,name=value,proto3" json:"value,omitempty"`
}

func (x *Number2) Reset() {
	*x = Number2{}
	if protoimpl.UnsafeEnabled {
		mi := &file_bin_protos_summator_proto_msgTypes[1]
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		ms.StoreMessageInfo(mi)
	}
}

func (x *Number2) String() string {
	return protoimpl.X.MessageStringOf(x)
}

func (*Number2) ProtoMessage() {}

func (x *Number2) ProtoReflect() protoreflect.Message {
	mi := &file_bin_protos_summator_proto_msgTypes[1]
	if protoimpl.UnsafeEnabled && x != nil {
		ms := protoimpl.X.MessageStateOf(protoimpl.Pointer(x))
		if ms.LoadMessageInfo() == nil {
			ms.StoreMessageInfo(mi)
		}
		return ms
	}
	return mi.MessageOf(x)
}

// Deprecated: Use Number2.ProtoReflect.Descriptor instead.
func (*Number2) Descriptor() ([]byte, []int) {
	return file_bin_protos_summator_proto_rawDescGZIP(), []int{1}
}

func (x *Number2) GetValue() float32 {
	if x != nil {
		return x.Value
	}
	return 0
}

var File_bin_protos_summator_proto protoreflect.FileDescriptor

var file_bin_protos_summator_proto_rawDesc = []byte{
	0x0a, 0x19, 0x62, 0x69, 0x6e, 0x2f, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x73, 0x2f, 0x73, 0x75, 0x6d,
	0x6d, 0x61, 0x74, 0x6f, 0x72, 0x2e, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x22, 0x39, 0x0a, 0x07, 0x4e,
	0x75, 0x6d, 0x62, 0x65, 0x72, 0x31, 0x12, 0x16, 0x0a, 0x06, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x31,
	0x18, 0x01, 0x20, 0x01, 0x28, 0x02, 0x52, 0x06, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x31, 0x12, 0x16,
	0x0a, 0x06, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x32, 0x18, 0x02, 0x20, 0x01, 0x28, 0x02, 0x52, 0x06,
	0x76, 0x61, 0x6c, 0x75, 0x65, 0x32, 0x22, 0x1f, 0x0a, 0x07, 0x4e, 0x75, 0x6d, 0x62, 0x65, 0x72,
	0x32, 0x12, 0x14, 0x0a, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x18, 0x01, 0x20, 0x01, 0x28, 0x02,
	0x52, 0x05, 0x76, 0x61, 0x6c, 0x75, 0x65, 0x32, 0x27, 0x0a, 0x08, 0x53, 0x75, 0x6d, 0x6d, 0x61,
	0x74, 0x6f, 0x72, 0x12, 0x1b, 0x0a, 0x03, 0x73, 0x75, 0x6d, 0x12, 0x08, 0x2e, 0x4e, 0x75, 0x6d,
	0x62, 0x65, 0x72, 0x31, 0x1a, 0x08, 0x2e, 0x4e, 0x75, 0x6d, 0x62, 0x65, 0x72, 0x32, 0x22, 0x00,
	0x42, 0x0a, 0x5a, 0x08, 0x2e, 0x2f, 0x70, 0x72, 0x6f, 0x74, 0x6f, 0x73, 0x62, 0x06, 0x70, 0x72,
	0x6f, 0x74, 0x6f, 0x33,
}

var (
	file_bin_protos_summator_proto_rawDescOnce sync.Once
	file_bin_protos_summator_proto_rawDescData = file_bin_protos_summator_proto_rawDesc
)

func file_bin_protos_summator_proto_rawDescGZIP() []byte {
	file_bin_protos_summator_proto_rawDescOnce.Do(func() {
		file_bin_protos_summator_proto_rawDescData = protoimpl.X.CompressGZIP(file_bin_protos_summator_proto_rawDescData)
	})
	return file_bin_protos_summator_proto_rawDescData
}

var file_bin_protos_summator_proto_msgTypes = make([]protoimpl.MessageInfo, 2)
var file_bin_protos_summator_proto_goTypes = []interface{}{
	(*Number1)(nil), // 0: Number1
	(*Number2)(nil), // 1: Number2
}
var file_bin_protos_summator_proto_depIdxs = []int32{
	0, // 0: Summator.sum:input_type -> Number1
	1, // 1: Summator.sum:output_type -> Number2
	1, // [1:2] is the sub-list for method output_type
	0, // [0:1] is the sub-list for method input_type
	0, // [0:0] is the sub-list for extension type_name
	0, // [0:0] is the sub-list for extension extendee
	0, // [0:0] is the sub-list for field type_name
}

func init() { file_bin_protos_summator_proto_init() }
func file_bin_protos_summator_proto_init() {
	if File_bin_protos_summator_proto != nil {
		return
	}
	if !protoimpl.UnsafeEnabled {
		file_bin_protos_summator_proto_msgTypes[0].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*Number1); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
		file_bin_protos_summator_proto_msgTypes[1].Exporter = func(v interface{}, i int) interface{} {
			switch v := v.(*Number2); i {
			case 0:
				return &v.state
			case 1:
				return &v.sizeCache
			case 2:
				return &v.unknownFields
			default:
				return nil
			}
		}
	}
	type x struct{}
	out := protoimpl.TypeBuilder{
		File: protoimpl.DescBuilder{
			GoPackagePath: reflect.TypeOf(x{}).PkgPath(),
			RawDescriptor: file_bin_protos_summator_proto_rawDesc,
			NumEnums:      0,
			NumMessages:   2,
			NumExtensions: 0,
			NumServices:   1,
		},
		GoTypes:           file_bin_protos_summator_proto_goTypes,
		DependencyIndexes: file_bin_protos_summator_proto_depIdxs,
		MessageInfos:      file_bin_protos_summator_proto_msgTypes,
	}.Build()
	File_bin_protos_summator_proto = out.File
	file_bin_protos_summator_proto_rawDesc = nil
	file_bin_protos_summator_proto_goTypes = nil
	file_bin_protos_summator_proto_depIdxs = nil
}


```

```

package calculations

func Summator(x float32, y float32) float32 {
	z := x + y
	return z
}
```

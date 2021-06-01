# 样例代码
## 创建如下工程结构
![image](https://user-images.githubusercontent.com/41630875/120259230-241b1580-c2c6-11eb-9d6e-67dd6c48d4c0.png)


## 各文件代码如下
切记不要删除注释，注释为code-generator.sh所需信息
* 顶层register.go
```go
package bolingcavalry

const (
	GroupName = "bolingcavalry.k8s.io"
	Version   = "v1"
)
```
* doc.go
```go
// +k8s:deepcopy-gen=package

// +groupName=bolingcavalry.k8s.io
package v1
```
* register.go
```go
package v1

import (
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/runtime"
	"k8s.io/apimachinery/pkg/runtime/schema"

	"CustomK8SResource/pkg/apis/bolingcavalry"
)

var SchemeGroupVersion = schema.GroupVersion{
	Group:   bolingcavalry.GroupName,
	Version: bolingcavalry.Version,
}

var (
	SchemeBuilder = runtime.NewSchemeBuilder(addKnownTypes)
	AddToScheme   = SchemeBuilder.AddToScheme
)

func Resource(resource string) schema.GroupResource {
	return SchemeGroupVersion.WithResource(resource).GroupResource()
}

func Kind(kind string) schema.GroupKind {
	return SchemeGroupVersion.WithKind(kind).GroupKind()
}

func addKnownTypes(scheme *runtime.Scheme) error {
	scheme.AddKnownTypes(
		SchemeGroupVersion,
		&Student{},
		&StudentList{},
	)

	// register the type in the scheme
	metav1.AddToGroupVersion(scheme, SchemeGroupVersion)
	return nil
}
```
* types.go
```go
package v1

import (
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
)

// +genclient
// +genclient:noStatus
// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object

type Student struct {
	metav1.TypeMeta   `json:",inline"`
	metav1.ObjectMeta `json:"metadata,omitempty"`
	Spec              StudentSpec `json:"spec"`
}

type StudentSpec struct {
	name   string `json:"name"`
	school string `json:"school"`
}

// +k8s:deepcopy-gen:interfaces=k8s.io/apimachinery/pkg/runtime.Object

// StudentList is a list of Student resources
type StudentList struct {
	metav1.TypeMeta `json:",inline"`
	metav1.ListMeta `json:"metadata"`

	Items []Student `json:"items"`
}
```

# 生成k8s代码
## 安装依赖
```shell
go get -u k8s.io/apimachinery/pkg/apis/meta/v1`
go get -u k8s.io/code-generator/...`
```

## 生成代码
将上述源码整个工程拷贝到$GOPATH/src下，进入工程根路径，在根路径执行以下命令
```shell
$GOPATH/pkg/mod/k8s.io/code-generator@v0.21.1/generate-groups.sh all \
CustomK8SResource/pkg/generated \
CustomK8SResource/pkg/apis \
bolingcavalry:v1 \
--go-header-file $GOPATH/pkg/mod/k8s.io/code-generator@v0.21.1/hack/boilerplate.go.txt \
--output-base $GOPATH/src/ \
-v 10
```

稍等片刻，最后打印如下日志即代表成功

![image](https://user-images.githubusercontent.com/41630875/120259105-e61df180-c2c5-11eb-8eaf-ffe83f27af80.png)

可见如下工程结构

![image](https://user-images.githubusercontent.com/41630875/120259191-106faf00-c2c6-11eb-80c6-862967e1e25c.png)


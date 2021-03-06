# dynamic form
对客户端原生做一个动态的表单下发，表单类型如下：

- 文本类型：客户端需传值string
- 纯数字：客户端需int转string提交
- 长文本：客户端需点击跳转至长文本编辑界面，并传值string
- 图片：客户端需调用上传接口，并传值string（图片url或其它标识）
- 日期：客户端需创建日期选择交互，并传值int转string（时间戳）
- 单选框：客户端需做单选框交互，并传值选择结果ID转string
- 复选框：客户端需做复选框交互，并传值选择结果列表转json string
- 子级输入框列表：客户端需点击跳转至子级输入框列表，传值根据各个子级类型而定

## 实现目标
实现一个通用的、各个业务均可直接调用的表单下发方案。存储在数据表中，区分不同业务ID及模块ID进行分别下发。
业务ID由具体实现接口传递，模块ID由客户端传递，表单下发服务具体提供以下参数：
```go
package formsvc
// check_type字段：校验用户输入类型(新类型从最后一行追加)
const (
	CheckTypeNull  = iota // 无
	CheckTypeNum          // 纯数字
	CheckTypeEmail        // 邮箱
	CheckTypeLink         // 链接
	CheckTypePhone		  // 电话号码
)

// input_type字段：用户输入类型(新类型从最后一行追加，客户端传值统一字符串)
const (
	InputTypeText    = iota // 文本类型(客户端传值后处理为string)
	InputTypeNum            // 纯数字(客户端传值后处理为int64)
	InputTypeLong           // 长文本，客户端需要跳转至长文本编辑界面(客户端传值后处理为string)
	InputTypeImage          // 图片，客户端需要做上传图片交互(客户端传图片url值后处理为string)
	InputTypeDate           // 日期，客户端需要做日期选择(客户端传值时间戳后处理为int64)
	InputTypeSin            // 单选框，客户端需要做下拉框选择(客户端传选项id值后处理为int8)
	InputTypeMul            // 复选框，客户端需要做下拉框选择(客户端传选项id列表后处理为int8[])
	InputTypeChiL           // 子集输入框，客户端需要跳转再做表单处理(客户端传值：按照以上类型处理)
	InputTypeSdEmail        // 发送邮件，用于验证码，客户端需要请求邮件发送接口(客户端传值后处理为string)
	InputTypeCountry        // 输入国家，会下发不同于单选框的选项列表(客户端传值后处理为string)
)

// is_required字段：是否必填
const (
	RequiredTrue  = 1 // 必填
	RequiredFalse = 0 // 非必填
)

// SizeRange 大小范围
type SizeRange struct {
	Min int64 `json:"min"`
	Max int64 `json:"max"`
}

// SubOption 子选项
type SubOption struct {
	ID   int64  `json:"id"`
	Name string `json:"name"`
}

// SubCountry 子国家选项
type SubCountry struct {
	HotCountries []string `json:"hot_countries"`
	Countries    []string `json:"countries"`
}

// FormConfigItem 结果元素
type FormConfigItem struct {
	ID         uint64 			 `json:"id"`
	Name       string 			 `json:"name"`
	InputType  int   			 `json:"input_type"`
	IsRequired int8   			 `json:"is_required"`
	// 返回时为nil，业务所需时自行填充
	Default    string 			 `json:"default"`
	// 该输入框最终最终添加到数据表中对应的字段名
	TableName  string			 `json:"table_name,omitempty"`
	FieldName  string            `json:"field_name,omitempty"`
	Tag 	   string			 `json:"tag,omitempty"`
	CheckType  int               `json:"check_type"`
	SizeRange  *SizeRange        `json:"size_range"`
	Hint       string            `json:"hint"`
	ErrMsg     string            `json:"err_msg"`
	SubOption  []*SubOption      `json:"sub_option"`
	SubCountry *SubCountry       `json:"sub_country"`
	SubInput   []*FormConfigItem `json:"sub_input"`
}
```

## 具体交互
客户端请求具体不同业务的表单下发接口，携带自身业务的business_id(业务ID)、module_id(同一业务下的模块ID)，而showcase_id由服务端根据不同业务区分下发。
客户端进行表单提交时，取各个输入框的ID与input_type对应的传值，form表单内为自定义字段和items字段(多项map构成的数组json序列化后提交)，如：
```json
{
    "id":123,
    "items":[
        {
            "id":1,
            "data":"3500"
        },
        {
            "id":2,
            "data":"800"
        },
        {
            "id":3,
            "data":"969520393"
        },
        {
            "id":4,
            "data":"caoyang@stary.itd"
        },
        {
            "id":5,
            "data":"xxddww"
        },
        {
            "id":6,
            "data":"www.dreame.com"
        }
    ]
}
```
数据表设计见sql目录

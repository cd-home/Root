[TOC]

### Standard library and Packages(标准库)

#### Init()

考点：特殊函数init；无参数、无返回值; 一个包可以有多个init函数; 初始化顺序由依赖包的顺序决定

~~~go
func init() {
    
}
~~~

#### JSON unmarshalling

考点：JSON序列化与反序列化; 结构体成员对外可见性；json-tag的配合

~~~go
type Result struct {
	Status int `json:"status"`
}
func main() {
	data := []byte(`{"status": 200}`)
	result := Result{}

	if err := json.Unmarshal(data, &result); err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("result=%+v", result)
}
~~~

#### Utf8 length

考点：utf8长度；英文、中文

~~~go
func main() {
	fmt.Println(len("A"))    // 1 一个英文1个字节
	fmt.Println(len("你好"))  // 6 一个汉字占三个字节
}

~~~

#### Context.WithTimeout

考点：contex包超时控制

~~~go
func main() {
	timeout := 3 * time.Second
	ctx, cancelFunc := context.WithTimeout(
        context.Background(), 
        timeout)
	defer cancelFunc()
	select {
	case <-time.After(time.Second):
		fmt.Println(1)
	case <-time.After(2 * time.Second):
		fmt.Println(2)
	case <-time.After(3 * time.Second):
		fmt.Println(3)
	case <-ctx.Done():
		fmt.Println(ctx.Err())
	}
}
~~~

#### Flag

考点：命令行参数的获取

~~~go
var ip string
var port int64

func init()  {
	flag.StringVar(&ip, "ip", "127.0.0.1", "ip address")
	flag.Int64Var(&port, "port", 8080, "port")
}

func main() {
	flag.Parse()
	fmt.Printf("%s:%d", ip, port)
}
~~~

#### Http Server

考点：HTTP服务器搭建

~~~go
func Hello(w http.ResponseWriter, r *http.Request) {
	fmt.Println(r.URL.Host)
	fmt.Fprintf(w, "Hello World")
}
func main() {
	http.HandleFunc("/", Hello)
	http.ListenAndServe(":8080", nil)
}
~~~

#### Sql query

database包的适用

~~~go
import (
	"database/sql"
	_ "github.com/go-sql-driver/mysql"
)

func main() {
    DB, err := sql.Open("mysql", "url") // 假设已经建立了连接
	// 查询多条
	data := make([]map[string]xxstruct{}, 0)
	rows, err := DB.Query()
	rows.Scan()
    // 每次 db.Query 操作后，都建议调⽤用 rows.Close()
	defer rows.Close()    
	for rows.Next() {
		rows.Scan()
	 //.....  扫描出来的可以以结构体追加到切片，或者扫描为map追加到切片
	 data = append(data, temp)
	}
}
~~~
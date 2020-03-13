# 理解 go 的 sort

sort 包 在内部实现了四种基本的排序算法：插入排序（insertionSort）、归并排序（symMerge）、堆排序（heapSort）和快速排序（quickSort）； sort 包会依据实际数据自动选择最优的排序算法。所以我们写代码时只需要考虑实现 sort.Interface 这个类型就可以了。
<!--more -->

## 源码
源码包 sort
sort 包定义了Interface接口，包含一下三个方法，只要实现了这个接口，就可以调用sort包中的排序方法。

```golang
type Interface interface {
    // Len is the number of elements in the collection.
    Len() int
    // Less reports whether the element with
    // index i should sort before the element with index j.
    Less(i, j int) bool
    // Swap swaps the elements with indexes i and j.
    Swap(i, j int)
}
```

先看看 sort 包本身对于 []int 类型如何排序
```go
# sort/sort.go
// IntSlice attaches the methods of Interface to []int, sorting in increasing order.
type IntSlice []int

func (p IntSlice) Len() int           { return len(p) }
func (p IntSlice) Less(i, j int) bool { return p[i] < p[j] }
func (p IntSlice) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }

// Sort is a convenience method.
func (p IntSlice) Sort() { Sort(p) }
```
IntSlice 使用的是 快速排序

``` go
# sort/sort.go
// Sort sorts data.
// It makes one call to data.Len to determine n, and O(n*log(n)) calls to
// data.Less and data.Swap. The sort is not guaranteed to be stable.
func Sort(data Interface) {
	n := data.Len()
	quickSort(data, 0, n, maxDepth(n))
}
```
>sort 包 在内部实现了四种基本的排序算法：插入排序（insertionSort）、归并排序（symMerge）、堆排序（heapSort）和快速排序（quickSort）； sort 包会依据实际数据自动选择最优的排序算法。所以我们写代码时只需要考虑实现 sort.Interface 这个类型就可以了。

sort.go文件中还定义了 IsSorted() Reverse() 等方法，只要实现了 Interface 接口，就可以调用这些方法。
## 例子
```go
type TDepartment struct {
	Ctx          context.Context `json:"-" db:"-"`
	DeptId       int64           `json:"f_dept_id" db:"f_dept_id"`
	CorpId       int64           `json:"f_corp_id" db:"f_corp_id"`
	ParentDeptId int64           `json:"f_parent_dept_id" db:"f_parent_dept_id"`
	Sort         int32           `json:"f_sort" db:"f_sort"`
	DeptName     string          `json:"f_dept_name" db:"f_dept_name"`
	CreateTime   string          `json:"f_create_time" db:"f_create_time"`
	ModifyTime   string          `json:"f_modify_time" db:"f_modify_time"`
}
type TDepartmentSlice []*TDepartment

func (p TDepartmentSlice) Len() int { return len(p) }

// 根据DeptId 排序
func (p TDepartmentSlice) Less(i, j int) bool {
	return p[i].DeptId < p[j].DeptId
}
func (p TDepartmentSlice) Swap(i, j int) { p[i], p[j] = p[j], p[i] }

type SortTDepartmentSliceByCreateTime struct {
	TDepartmentSlice
}

func (this SortTDepartmentSliceByCreateTime) Less(i, j int) bool {
	t1, _ := time.Parse("2006-01-02 15:04:05", this.TDepartmentSlice[i].CreateTime)
	t2, _ := time.Parse("2006-01-02 15:04:05", this.TDepartmentSlice[j].CreateTime)
	return t1.Unix() < t2.Unix()
}

//func (this TDepartmentSlice) Swap(i, j int) { this[i], this[j] = this[j], this[i] }

func main() {
	t1 := TDepartment{
		DeptId:     1,
		CreateTime: "2019-08-10 15:11:12",
	}
	t2 := TDepartment{
		DeptId:     2,
		CreateTime: "2019-08-10 15:11:23",
	}
	t3 := TDepartment{
		DeptId:     4,
		CreateTime: "2019-08-18 15:11:12",
	}
	t4 := TDepartment{
		DeptId:     3,
		CreateTime: "2019-08-08 15:11:12",
	}
	penson := TDepartmentSlice{}
	penson = append(penson, &t1, &t2, &t3, &t4)
	for _, v := range penson {
		fmt.Println(fmt.Sprintf("deptId:%v.createTime:%v", v.DeptId, v.CreateTime))
	}
	sort.Sort(TDepartmentSlice(penson))
	for _, v := range penson {
		fmt.Println(fmt.Sprintf("deptId:%v.createTime:%v", v.DeptId, v.CreateTime))
	}
	sort.Sort(SortTDepartmentSliceByCreateTime{penson})
	for _, v := range penson {
		fmt.Println(fmt.Sprintf("deptId:%v.createTime:%v",v.DeptId,v.CreateTime))
	}
}
```
## 总结
- 如果要对slice实现排序，就需要实现 Interface接口
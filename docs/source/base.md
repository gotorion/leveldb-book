# 基础组件
leveldb中有些类设计比较小但是又比较通用，在此统一介绍。
## Slice
`Slice`类是对字符串切片的简单封装，内部成员如下：

```cpp
class Slice {
private:
   const char* data_;
   size_t size_;
};
```
由于`Slice`本质上是一个胖指针，空间上只占用一个指针类型和size_t类型的大小，可以避免在传递字符串时的大量拷贝。C++17的`string_view`或C++20的`span`作用类似，但是由于leveldb开发较早，无法使用这些新特性。

使用`Slice`类时要特别注意，由于`Slice`并没有数据的所有权，需要注意指向对象的生命周期。如果指向的数据生命周期在`Slice`析构之前结束会导致Undefined Behavior。
## Status
`Status`类是在leveldb中用于接口返回，其成员如下：
```cpp
class Status {
public:
   bool ok() const { return (state_ == nullptr); }
   // Return a success status.
   static Status OK() { return Status(); 
private:
   const char * state_;
};
```
当`state_ == nullptr`时表示成功状态，否则在内部定义了`IOError`，`NotFound`等类型的错误码，用`state_[0..3]`表示错误消息的长度，`state_[4]`表示以上错误码，`state_[5..]`用于存储错误消息。这样设计可以保证接口返回时只需拷贝一个指针大小即可获得全部的错误信息。
笔者之前项目的错误处理方式是使用枚举类型的错误码，每次获取错误信息还需要查找全局的map获取对应字符串，实在不优雅。

## Filter

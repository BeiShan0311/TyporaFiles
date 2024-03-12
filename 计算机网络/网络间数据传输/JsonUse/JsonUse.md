```c
#include <stdio.h>
#include <json-c/json.h>

#define BUFFER_SIZE 16

// 将{"type":1, "name":"zhou", "passwd":123, "books":["book1", "book2", "book3"]}封装为Json对象
int main()
{
    // 1.创建JSON对象
    struct json_object *object = json_object_new_object();
    if (object == NULL)
    {
        perror("null pointer error");
        exit(-1);
    }

    // 在json对象中添加<key, value>元素
    json_object_object_add(object, "type", json_object_new_int(1));
    json_object_object_add(object, "name", json_object_new_string("zhou"));
    json_object_object_add(object, "passwd", json_object_new_int64(123));

    struct json_object *arrayBooks = json_object_new_array();
    json_object_array_add(arrayBooks, json_object_new_string("C++"));
    json_object_array_add(arrayBooks, json_object_new_string("PHP"));
    json_object_object_add(object, "books", arrayBooks);

    // 将json对象转化为字符串
    const char *str = json_object_to_json_string(object);
    printf("%s\n", str);

    // 将json类型的字符串解析出来
    // 服务器代码
    // 1.将json类型字符串先解析成json对象

    struct json_object *parse_object = json_tokener_parse(str);

    printf("type:%d\n", json_object_get_int(json_object_object_get(parse_object, "type")));
    printf("name:%s\n", json_object_get_string(json_object_object_get(parse_object, "name")));
    printf("passwd:%s\n", json_object_get_string(json_object_object_get(parse_object, "passwd")));
    
    struct json_object * booksList = json_object_object_get(parse_object, "books");
    // 数组长度
    size_t size = json_object_array_length(booksList);
    printf("size:%ld\n", size);

    for (int idx = 0; idx < size; idx++)
    {
        printf("books[%d] = %s\n", idx, json_object_get_string(json_object_array_get_idx(booksList, idx)));
    }

    return 0;
}
```


# Newtonsoft.Json高级用法之枚举中文转义

现假设有枚举类型
```
public enum NotifyType
    {
        /// <summary>
        /// Emil发送
        /// </summary>
        Mail=0,

        /// <summary>
        /// 短信发送
        /// </summary>
        SMS=1
    }

public class TestEnum
    {
        /// <summary>
        /// 消息发送类型
        /// </summary>
        public NotifyType Type { get; set; }
    }

    JsonConvert.SerializeObject(new TestEnum());
```
输出结果：
```
{
    "Type":0
}
```
如果想要输出对应的英文描述如 Mail,SMS，可以借助Newtonsoft.Json内置的转换类型 *StringEnumConverter*
```
public class TestEnum
    {
        /// <summary>
        /// 消息发送类型
        /// </summary>
        [JsonConverter(typeof(StringEnumConverter))]
        public NotifyType Type { get; set; }
    }
```
则输出结果：
```
{
    "Type":"Mail"
}
```
那如果想要实现输出中文描述如电子邮箱、短信呢，则需要我们自己去实现相应的转换器
首先需要改造枚举类型，借助 *Description* 属性添加中文描述
```
public enum NotifyType
{
            /// <summary>
            /// 电子邮箱
            /// </summary>
            [Description("电子邮箱")]
            Mail = 0,

            /// <summary>
            /// 手机短信
            /// </summary>
            [Description("手机短信")] 
            SMS = 1 
}
```
其次，创建描述转换器 *DescriptionEnumConverter*
```
public class DescriptionEnumConverter : JsonConverter
    {
        public override bool CanConvert(Type objectType)
        {
            return true;
        }

        public override object ReadJson(JsonReader reader, Type objectType, object existingValue, JsonSerializer serializer)
        {
            if (reader.TokenType == JsonToken.Null)
            {
                return null;
            }

            try
            {
                return reader.Value.ToString();
            }
            catch (Exception)
            {
                throw new Exception(string.Format("不能将枚举{1}的值{0}转换为Json格式.", reader.Value, objectType));
            }
        }

        public override void WriteJson(JsonWriter writer, object value, JsonSerializer serializer)
        {
            if (value == null)
            {
                writer.WriteNull();
                return;
            }
            string bValue = value.ToString();
            int isNo;
            if (int.TryParse(bValue, out isNo))
            {
                bValue = GetEnumDescription(value.GetType(), isNo);
            }
            else
            {
                bValue = GetEnumDescription(value.GetType(), value.ToString());
            }


            writer.WriteValue(bValue);
        }

        /// <summary>
        /// 获取枚举描述
        /// </summary>
        /// <param name="type">枚举类型</param>
        /// <param name="value">枚举名称</param>
        /// <returns></returns>
        private string GetEnumDescription(Type type, string value)
        {
            try
            {
                FieldInfo field = type.GetField(value);
                if (field == null)
                {
                    return string.Empty;
                }

                var desc = Attribute.GetCustomAttribute(field, typeof(DescriptionAttribute)) as DescriptionAttribute;
                if (desc != null) return desc.Description;

                return string.Empty;
            }
            catch
            {
                return string.Empty;
            }
        }

        /// <summary>
        /// 获取枚举描述
        /// </summary>
        /// <param name="type">枚举类型</param>
        /// <param name="value">枚举hasecode</param>
        /// <returns></returns>
        private string GetEnumDescription(Type type, int value)
        {
            try
            {

                FieldInfo field = type.GetField(Enum.GetName(type, value));
                if (field == null)
                {
                    return string.Empty;
                }

                var desc = Attribute.GetCustomAttribute(field, typeof(DescriptionAttribute)) as DescriptionAttribute;
                if (desc != null) return desc.Description;

                return string.Empty;
            }
            catch
            {
                return string.Empty;
            }
        }
    }
```
最后，在类定义上指定该属性使用上一步创建的转换器：
```
public class TestEnum
    {
        /// <summary>
        /// 消息发送类型
        /// </summary>
        [JsonConverter(typeof(DescriptionEnumConverter))]
        public NotifyType Type { get; set; }
    }
```

此时即可得到输出结果：
```
{
    "Type":"电子邮箱"
}
```
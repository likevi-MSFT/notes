Code:
```C#
namespace TestConsoleApp
{
    using System;
    using Newtonsoft.Json;
    using Newtonsoft.Json.Linq;
    using Newtonsoft.Json.Serialization;

    #region Generic

    public class Description<Properties>
        where Properties : PropertyBase
    {
        public Description()
        {

        }

        public string Type { get; set; }

        public Properties P { get; set; }
    }

    public sealed class PropertyA : PropertyBase
    {
        public static string Type = "A";

        [JsonConstructor]
        public PropertyA()
        {

        }

        public string S { get; set; }
    }

    public sealed class PropertyB : PropertyBase
    {
        public static string Type = "B";

        [JsonConstructor]
        public PropertyB()
        {

        }

        public int I { get; set; }
    }

    public abstract class PropertyBase
    {

    }

    public class DescriptionConverter : JsonConverter
    {
        public override bool CanConvert(Type objectType)
        {
            return objectType.IsAssignableFrom(typeof(Description<PropertyBase>));
        }

        public override bool CanWrite => false;

        public override object ReadJson(JsonReader reader, Type objectType, object existingValue, JsonSerializer serializer)
        {
            JObject jobject = JObject.Load(reader);
            var type = jobject.GetValue("type");

            if (type.Value<string>() == PropertyA.Type)
            {
                var inter = jobject.ToObject<Description<PropertyA>>(serializer);
                return new Description<PropertyBase>()
                {
                    Type = inter.Type,
                    P = inter.P
                };
            }
            else if (type.Value<string>() == PropertyB.Type)
            {
                var inter = jobject.ToObject<Description<PropertyB>>(serializer);
                return new Description<PropertyBase>()
                {
                    Type = inter.Type,
                    P = inter.P
                };
            }

            throw new Exception("Um, this is some other type!");
        }

        public override void WriteJson(JsonWriter writer, object value, JsonSerializer serializer)
        {
            throw new NotImplementedException();
        }
    }

    #endregion

    public class Program
    {
        static void Main(string[] args)
        {
            var description = new Description<PropertyA>()
            {
                Type = PropertyA.Type,
                P = new PropertyA()
                {
                    S = "some string"
                }
            };

            var serializerSettings = new JsonSerializerSettings()
            {
                StringEscapeHandling = StringEscapeHandling.EscapeHtml,
                ContractResolver = new CamelCasePropertyNamesContractResolver(),
                NullValueHandling = NullValueHandling.Ignore,
            };
            serializerSettings.Converters.Add(new DescriptionConverter());

            var descriptionJson = JsonConvert.SerializeObject(description, serializerSettings);

            var descriptionD = JsonConvert.DeserializeObject<Description<PropertyBase>>(descriptionJson, serializerSettings);

            if (descriptionD.Type == PropertyA.Type)
            {
                var descriptionDA = new Description<PropertyA>()
                {
                    Type = PropertyA.Type,
                    P = (PropertyA)descriptionD.P,
                };
            }

            var descriptionDJson = JsonConvert.SerializeObject(descriptionD, serializerSettings);
        
            Console.WriteLine($"Serialized description, descriptionJson    {descriptionJson}");
            Console.WriteLine($"Serialized deserialized description json   {descriptionDJson}");
        }
    }
}
```

Output
```
Serialized description, descriptionJson    {"type":"A","p":{"s":"some string"}}
Serialized deserialized description json   {"type":"A","p":{"s":"some string"}}
```

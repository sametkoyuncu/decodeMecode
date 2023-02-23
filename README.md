# decodeMecode
Bu durumda, ResponseData modelimizi biraz değiştirmemiz gerekiyor. data alanı için bir enum tanımlayabiliriz ve bu enum, olası tüm değerleri içerecektir (text, button, htmlText gibi). Bu sayede, JSON'daki farklı veri tiplerini tek bir modele bağlayabiliriz.

Aşağıdaki gibi bir kod örneği ile ConversationData modelini güncelleyebiliriz:

```swift
struct ConversationData: Decodable {
    let conversationId: String
    let response: [[ResponseData]]
    
    enum CodingKeys: String, CodingKey {
        case conversationId = "conversation_id"
        case response
    }
}

struct ResponseData: Decodable {
    let type: DataType
    let data: DataValue
    
    enum DataType: String, Decodable {
        case text
        case button
        case htmlText = "htmlText"
    }
    
    enum DataValue: Decodable {
        case text(String)
        case button([ButtonData])
        case htmlText([HtmlData])
        
        init(from decoder: Decoder) throws {
            let container = try decoder.singleValueContainer()
            if let text = try? container.decode(String.self) {
                self = .text(text)
            } else if let buttons = try? container.decode([ButtonData].self) {
                self = .button(buttons)
            } else if let html = try? container.decode([HtmlData].self) {
                self = .htmlText(html)
            } else {
                throw DecodingError.typeMismatch(DataValue.self, DecodingError.Context(codingPath: decoder.codingPath, debugDescription: "Invalid data type"))
            }
        }
    }
}

struct ButtonData: Decodable {
    let payload: String
    let title: String
}

struct HtmlData: Decodable {
    let attrs: [String: String]
    let tag: String
    let text: String
}
```

Bu modelde, ResponseData öğeleri, type ve data alanlarına sahip olur. type alanı, text, button veya htmlText olabilir ve data alanı, DataValue enum'ından bir değer alır. DataValue enum'u, farklı veri tiplerini kapsayan bir örtüşme sağlar (text, button ve htmlText). init(from:) metodunda, decoder'dan tek bir değer alınarak, veri tipi kontrol edilir ve uygun şekilde bir DataValue oluşturulur. Bu şekilde, değişken veri tipleri ile başa çıkabiliriz.

Bu model, örnek veriyi doğru şekilde decode etmek için kullanılabilir:

```swift
let decoder = JSONDecoder()
do {
    let conversationData = try decoder.decode(ConversationData.self, from: data)
    print(conversationData)
} catch {
    print(error)
}
```

conversationData modelindeki response öğesi, farklı tiplerdeki verileri (text, button, htmlText) içerebilir ve modelimiz bu farklı veri tipleri ile başa çıkabilir.

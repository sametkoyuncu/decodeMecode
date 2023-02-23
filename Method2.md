```swift
struct ConversationData: Decodable {
    let conversationId: String
    let response: [[ResponseData]]
    
    enum CodingKeys: String, CodingKey {
        case conversationId = "conversation_id"
        case response
    }
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        conversationId = try container.decode(String.self, forKey: .conversationId)
        
        var responseContainer = try container.nestedUnkeyedContainer(forKey: .response)
        var responseArray = [[ResponseData]]()
        
        while !responseContainer.isAtEnd {
            var innerResponseArray = [ResponseData]()
            let innerContainer = try responseContainer.nestedContainer(keyedBy: ResponseData.CodingKeys.self)
            
            while !innerContainer.isAtEnd {
                let responseData = try innerContainer.decode(ResponseData.self)
                innerResponseArray.append(responseData)
            }
            
            responseArray.append(innerResponseArray)
        }
        
        response = responseArray
    }
}

struct ResponseData: Decodable {
    let type: String
    let data: Any
    
    enum CodingKeys: String, CodingKey {
        case type
        case data
    }
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        type = try container.decode(String.self, forKey: .type)
        
        do {
            data = try container.decode(String.self, forKey: .data)
        } catch DecodingError.typeMismatch {
            do {
                data = try container.decode([ButtonData].self, forKey: .data)
            } catch DecodingError.typeMismatch {
                data = try container.decode([HtmlTextData].self, forKey: .data)
            }
        }
    }
}

struct ButtonData: Decodable {
    let payload: String
    let title: String
}

struct HtmlTextData: Decodable {
    let tag: String
    let text: String
    let attrs: [String: String]?
}

```

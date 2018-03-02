# Realm & Swift Codable
How to implement Swift 4 Codable with Realm Database in your Models?

This guide will demonstrate the implementation method for Realm Object containing a List, with Swift 4 Codable.

### JSON
```JSON
{
    "id": 732,
    "name": "Vendor Name",
    "logo": ".../thumb/missing.png",
    "kitchens": [
      {
        "id": 36,
        "name": "Sandwiches"
      },
      {
        "id": 37,
        "name": "Fast Food"
      }
    ]
  }
```
### Model

```Swift
import RealmSwift

class VendorsList : Object, Decodable {
    @objc dynamic var id : Int = 0
    @objc dynamic var name : String?
    @objc dynamic var logo : String?
    // Create your Realm List.
    var kitchensList = List<VendorKitchens>()
    
    override class func primaryKey() -> String? {
        return "id"
    }
    
    private enum CodingKeys: String, CodingKey {
        case id
        case name
        case logo
        // Set JSON Object Key
        case kitchensList = "kitchens"

    }
    
    public required convenience init(from decoder: Decoder) throws {
        self.init()
        let container = try decoder.container(keyedBy: CodingKeys.self)
        self.id = try container.decode(Int.self, forKey: .id)
        self.name = try container.decode(String.self, forKey: .name)
        self.logo = try container.decode(String.self, forKey: .logo)
        // Map your JSON Array response
        let kitchens = try container.decodeIfPresent([VendorKitchens].self, forKey: .kitchensList) ?? [VendorKitchens()]
        kitchensList.append(objectsIn: kitchens)
        
    }
    
}


class VendorKitchens : Object, Decodable {
    @objc dynamic var id : Int = 0
    @objc dynamic var name : String?
    
    override class func primaryKey() -> String? {
        return "id"
    }

    private enum CodingKeys: String, CodingKey {
        case id
        case name
    }
}
```
### URLSession
```Swift
guard let getUrl = URL(string: "https://Your_Endpoint/.../") else { return }
        URLSession.shared.dataTask(with: getUrl) { (data, response, error) in
            guard let data = data else { return }
            do {
                let decoder = JSONDecoder()
                let getData = try decoder.decode([VendorsList].self, from: data)
                    for data in getData {
                        print(data.id)
                        print(data.name)
                        print(data.logo)
                    }
                    for kitchen in data.kitchensList {
                        print(kitchen.name)
                    }
                }
            } catch let err {
                print("Err", err)
            }
            }.resume()
```

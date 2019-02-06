# Order Generation

```ruby

class OrderGenerator
  include Interactor::Organizer

  organize OrderGeneration::CreateMerchantOrder,
           OrderGeneration::CreateMerchantOrderItem,
           OrderGeneration::CreateOrder
end
```
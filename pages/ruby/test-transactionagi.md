# test-transaction

```ruby


  def deliver!
    ActiveRecord::Base.transaction do
      @delivery_order =
        DeliveryOrder.find_by(id: deliver_params[:delivery_order_id])
      cus***REMOVED***_type_order!
      in_transit_order!
      check_driver!
      delivery_actions
      delivery_order
    end
  end
# transaction-example

```ruby


  def call
    ActiveRecord::Base.transaction do
      product_pricing_params[:products].map do |product_params|
        @product = products.find(product_params[:id])
        @attrs = product_params[:product_pricings_attributes]
        create_product_pricings
      end
      result.uniq
    end
  end
  # end-wiki

  private

  # Create pricings for each passed product
  def create_product_pricings
    arrange_attrs
    @product.update!(product_pricings_attributes: [@attrs])
    @product.save!
    result << @product if @attrs["_destroy"].blank?
  end

  def arrange_attrs
    @attrs["business_id"] = @current_user.business.id
    @attrs["consumer_location_id"] =
      consumer_locations.find(@consumer_location_id)&.id
    @attrs["id"] = prev_pricing.id if prev_pricing.present?
  end

  def prev_pricing
    @product.product_pricings.find_by(@attrs.except(:price, :_destroy))
  end

  def consumer_locations
    ConsumerLocationsFinder.new({}, @current_user).with_current_business.records
  end
end
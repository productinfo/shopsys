# Upgrade Instructions for Services Removal

This article describes upgrade instructions for [#627 model service layer removal](https://github.com/shopsys/shopsys/pull/627).
Upgrade instructions are in a separate article because there is a lot of instructions and we don't want to jam UPGRADE.md.
Follow these instructions only if you upgrade from `v7.0.0-beta4` to `v7.0.0-beta5`.

We removed services (i.e. *Service classes - not to be interchanged with Symfony services) from our model and moved logic into more suitable places.
Following instructions tell you which method was moved where, so you can upgrade your code and also tests.

If you use these methods, change their calling appropriately:
- `CustomerService::edit(User $user, UserData $userData)`  
  -> `User::edit(UserData $userData, CustomerPasswordService $customerPasswordService)`
- `CustomerService::createDeliveryAddress(DeliveryAddressData $deliveryAddressData)`  
  -> `DeliveryAddressFactory::create(DeliveryAddressData $data)`
- `CustomerService::editDeliveryAddress(User $user, DeliveryAddressData $deliveryAddressData, DeliveryAddress $deliveryAddress = null)`  
  -> `User::editDeliveryAddress(DeliveryAddressData $deliveryAddressData, DeliveryAddressFactoryInterface $deliveryAddressFactory)`
- `CustomerService::changeEmail(User $user, $email, User $userByEmail = null)`  
  -> `User::changeEmail(string $email, ?self $userByEmail)`
- `CustomerService::create(UserData $userData, BillingAddress $billingAddress, DeliveryAddress $deliveryAddress = null, User $userByEmail = null)`  
  -> `UserFactory::create(UserData $userData, BillingAddress $billingAddress, ?DeliveryAddress $deliveryAddress, ?User $userByEmail)`
- `CustomerService::getAmendedByOrder(User $user, Order $order)`  
  -> `CustomerDataFactoryInterface::createAmendedCustomerDataByOrder(User $user, Order $order)`
- `AdministratorGridService::rememberGridLimit(Administrator $administrator, Grid $grid)`  
  -> `Administrator::rememberGridLimit(Grid $grid, AdministratorGridLimitFactoryInterface $administratorGridLimitFactory)`
- `AdministratorGridService::restoreGridLimit(Administrator $administrator, Grid $grid)`  
  -> `Administrator::restoreGridLimit(Grid $grid)`
- `AdministratorService::setPassword(Administrator $administrator, $password)`  
  -> `Administrator::setPassword(string $password, EncoderFactoryInterface $encoderFactory)`
- `AdministratorService::edit(AdministratorData $administratorData, Administrator $administrator, Administrator $administratorByUserName = null)`  
  -> `Administrator::edit(AdministratorData $administratorData, EncoderFactoryInterface $encoderFactory, ?self $administratorByUserName)`
- `AdministratorService::delete(Administrator $administrator, $adminCountExcludingSuperadmin)`  
  -> `Administrator::checkForDelete(TokenStorageInterface $tokenStorage, int $adminCountExcludingSuperadmin)`
- `OrderService::calculateTotalPrice(Order $order)`  
  -> `Order::calculateTotalPrice(OrderPriceCalculation $orderPriceCalculation)`
- `ProductService::getProductSellingPricesIndexedByDomainIdAndPricingGroupId(Product $product, array $pricingGroups)`  
  -> `ProductFacade::getAllProductSellingPricesIndexedByDomainId(Product $product)`
- `ProductService::sortProductsByProductIds(array $products, array $orderedProductIds)`  
  -> `ProductRepository::getSortedProductsByIds($domainId, PricingGroup $pricingGroup, array $sortedProductIds)`
- `ProductService::markProductForVisibilityRecalculation(Product $product)`  
  -> `Product::markForVisibilityRecalculation()`
- `ProductService::edit(Product $product, ProductData $productData)`  
  -> `Product::edit(ProductCategoryDomainFactoryInterface $productCategoryDomainFactory, ProductData $productData, ProductPriceRecalculationScheduler $productPriceRecalculationScheduler)`
- `ProductService::delete(Product $product)`  
  -> `Product::getProductDeleteResult()`
- `ProductService::recalculateInputPriceForNewVatPercent(Product $product, $productManualInputPrices, $newVatPercent)`  
  -> `ProductManualInputPrice::recalculateInputPriceForNewVatPercent($inputPriceType, $newVatPercent, BasePriceCalculation $basePriceCalculation, InputPriceCalculation $inputPriceCalculation)`
- `OrderService::getOrderDetailUrl(Order $order)`  
  -> `OrderUrlGenerator::getOrderDetailUrl(Order $order)`
- `OrderService:: createOrderProductInOrder(Order $order, Product $product, Price $productPrice)`  
  -> `Order::addProduct(Product $product, Price $productPrice, OrderProductFactoryInterface $orderProductFactory, Domain $domain, OrderPriceCalculation $orderPriceCalculation)`
- `OrderService::editOrder(Order $order, OrderData $orderData)`  
  -> `Order::edit(OrderData $orderData, OrderItemPriceCalculation $orderItemPriceCalculation, OrderProductFactoryInterface $orderProductFactory, OrderPriceCalculation $orderPriceCalculation)`
- `OrderStatusService::checkForDelete`  
  -> `OrderStatus::checkForDelete`
- `OrderStatusService::createOrderProductInOrder`  
  -> `OrderStatus::addProductToOrder`

Following classes have been removed:
- `AdministratorService`
- `AdministratorGridService`
- `CustomerService`
- `ProductService`
- `OrderService`
- `OrderStatusService`
- `FlagService`

Following methods have been removed:
- `User::setDeliveryAddress()`, use `User::editDeliveryAddress()` instead
- `Administrator::addGridLimit()`, use `Administrator::rememberGridLimit()` instead
- `Administrator::removeGridLimit()` as it was not used anywhere
- `Administrator::getLimitByGridId()` as it was not used anywhere
- `FlagService::create()`, use `FlagFactory::create()` instead
- `FlagService::edit()`, use `Flag::edit()` instead

Following classes changed constructors:
- `AdministratorFacade`
- `AdministratorGridFacade`
- `CachedBestsellingProductFacade`
- `CustomerFacade`
- `FlagFacade`
- `OrderItemFacade`
- `OrderProductFacade`
- `OrderMailService`
- `OrderEditResult`
- `OrderFacade`
- `ProductInputPriceFacade`
- `ProductPriceRecalculator`
- `ProductFacade`
- `UserFactory`

Following functions visibility was changed to `protected` as there is no need to use them from outside of objects:
- `Administrator::getGridLimit()`
- `Order::setTotalPrice()`

Follow also additional upgrade instructions:
- Change return type of `DeliveryAddressFactory::create()` to `?DeliveryAddress` as it now returns `null` when `addressFilled` is `false`
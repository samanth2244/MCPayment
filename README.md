# Apple Pay

Using Apple Pay via MCPayment Library

First you need to set the constants in the AppDelegate that were obtained from Global Pay

    MCDefaultAccessToken        = @"fc746bbb15047989e83e13c94d75487";
    MCDefaultGPUserId           = @"aptest";
    MCDefaultGPPassword         = @"Global2015";
    MCDefaultMerchantId         = @"merchant.comviva.applepay";
    MCDefaultCountryCode        = @"US";
    MCDefaultCurrencyCode       = @"USD";
    
Next intialize and allocate the MCPaymentManager

    MCPaymentManager *paymentManager = [[MCPaymentManager alloc]init];
  
  
  Then set the EnvironmentType to Testing or Production and Payment mode to purchase or pre- auth or subscription.
  
    [paymentManager setEnvironmentType:MCEnvironmentTypeTesting];
    
    [paymentManager setPaymentMode:MCPaymentTypePurchase];
    
    
    Then you need to create the MCPaymentRequest:
    
    MCPaymentRequest *request=[[MCPaymentRequest alloc]init];
    request.supportedNetworks = @[MCPaymentNetworkVisa,MCPaymentNetworkMasterCard,MCPaymentNetworkAmericanExpress];
    request.merchantCapabilities = MCMerchantCapability3DS;
    request.shippingType = MCShippingTypeDelivery;
    request.requiredBillingAddressFields = MCAddressFieldAll;
    request.requiredShippingAddressFields = MCAddressFieldAll;
    
    NSDecimalNumber *freeAmount = [NSDecimalNumber decimalNumberWithString:@"0.00"];
    MCPaymentShippingMethod *freeShipping = [MCPaymentShippingMethod shippingMethodWithIdentifier:@"free" detail:@"ships in 7 days" amount:freeAmount label:@"Free Shipping"];
    request.shippingMethods = @[freeShipping];
    
    MCPaymentSummaryItem *summaryItem = [MCPaymentSummaryItem summaryItemWithLabel:@"iPhone"    amount:[NSDecimalNumber decimalNumberWithString:amount]];
    MCPaymentSummaryItem *totalItem   = [MCPaymentSummaryItem summaryItemWithLabel:@"GlobalPay" amount:[NSDecimalNumber decimalNumberWithString:totalAmount]];
    request.paymentSummaryItems = @[summaryItem,totalItem];
    
    Then you need to pass this payment request to below method to present the payment sheet
    
    [paymentManager presentPaymentAuthorizationViewControllerWithPaymentRequest:request
                                        presentingController:self
                                        delegate:self
                                        completion:^(MCPaymentResponse *response , NSError *error)
        {
            //you will get error if user cancel's the request or required parameters are not set or for invalid payment request
            if (error)
            {
                NSLog(@"error response %@",error);
            }
            else
            {
                NSLog(@"response %@",response.responseMessage);
            }
    }];
        
Then you will get Response Data for success or failur in completion block and also you will get the user's shipping and billing address if requested while creating payment request.




#pragma mark MCPaymentManagerDelegate Methods

You will get this call back when the user selects a new shipping address in the payment sheet. Here you need to pass the required summary items and shipping methods for the new shipping address. Based on the address validation whether shipping is available for the new address, you need to pass the completion status like MCPaymentAuthorizationStatusInvalidShippingPostalAddress  or MCPaymentAuthorizationStatusInvalidShippingContact or Success or Failure. If you don't want to update the summaryitems and shipping methods just pass nil.

- (void)paymentAuthorizationViewController:(UIViewController *)controller
                  didSelectShippingAddress:(ABRecordRef)address
                                completion:(void (^)(MCPaymentAuthorizationStatus status, NSArray *shippingMethods, NSArray *summaryItems))completion
{
    completion(MCPaymentAuthorizationStatusSuccess,nil,nil);
}




You will get this call back when the user selects a new shipping method in the payment sheet and you need to pass the updated summary items for the new shipping method like updating shipping charges. If you don't want to update the summary items just pass nil.

- (void)paymentAuthorizationViewController:(UIViewController *)controller
                   didSelectShippingMethod:(MCPaymentShippingMethod *)shippingMethod
                                completion:(void (^)(MCPaymentAuthorizationStatus status, NSArray *summaryItems))completion
{
    completion(MCPaymentAuthorizationStatusSuccess,nil]);
}

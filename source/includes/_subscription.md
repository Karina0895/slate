# Subscription

The `Subscription` resource represents the subscriptions associated with a given customer `Account`. Such subscriptions are created by specifying a well defined `Plan` from the catalog, and an effective date at which to start. Such attributes -- among others -- , are used by the system to drive the invoice
generation. Aside from the billing aspect -- invoice generation --, Kill Bill also offers a parallel view to track the entitlement, i.e the service associated with the subscription, bascially answering the two following questions:

* Given a date, how much should we invoice ?
* Given a date, is the service available to the customer? 

For most of the use cases, those 2 views are one and the same, i.e the customer gets invoiced for what she consumes.

Subscriptions can be in one of the following state:

* `PENDING`: The subscription was created in the future
* `ACTIVE`: The subscription is currently active, its start date is in the past, its cancellation date -- if it exists -- is in the future, and it has not been paused.
* `BLOCKED`: The subscription is currently paused.
* `CANCELLED`: The subscription has been cancelled -- end state.

Please refer to our [subscription manual](http://docs.killbill.io/latest/userguide_subscription.html) for more details.

## Subscription Resource

The `Subscription` resource represent the subscription created by the user.

The attributes are the following:

* **`accountId`** <span style="color:#32A9C7">*[System generated, immutable]*</span>: The `ID` allocated by Kill Bill upon creation.
* **`bundleId`** <span style="color:#32A9C7">*[System generated, immutable]*</span>: The `ID` allocated by Kill Bill upon creation.
* **`subscriptionId`** <span style="color:#32A9C7">*[System generated, immutable]*</span>: The `ID` allocated by Kill Bill upon creation.
* **`externalKey`** <span style="color:#32A9C7">*[User generated]*</span>: The `Subscription` external key provided from client.
* **`bundleExternalKey`** <span style="color:#32A9C7">*[User generated]*</span>: The `Bundle` external key provided from client.
* **`startDate`** <span style="color:#32A9C7">*[User generated, immutable]*</span>: The (entitlement) start date -- when the service starts.
* **`productName`** <span style="color:#32A9C7">*[User generated]*</span>: The name of the active product catalog.
* **`productCategory`** <span style="color:#32A9C7">*[User generated]*</span>: The product category, either `BASE`, `ADD_ON` or `STANDALONE`.
* **`billingPeriod`** <span style="color:#32A9C7">*[User generated]*</span>: The billing period -- e.g `MONTHLY`.
* **`phaseType`** <span style="color:#32A9C7">*[User generated]*</span>: The type of the plan phase, either `TRIAL`, `DISCOUNT`, `FIXEDTERM`, `EVERGREEN`.   
* **`priceList`** <span style="color:#32A9C7">*[User generated]*</span>: The name of the active price list from the catalog.
* **`planName`** <span style="color:#32A9C7">*[User generated]*</span>: The name of the active plan catalog.
* **`state`** <span style="color:#32A9C7">*[System generated]*</span>: The state of the subscription.
* **`sourceType`** <span style="color:#32A9C7">*[System generated, immutable]*</span>: The kind of subscription -- e.g `NATIVE`
* **`cancelledDate`** <span style="color:#32A9C7">*[User generated]**</span>: The (entitlement) cancellation date -- when the service stops.
* **`chargedThroughDate`** <span style="color:#32A9C7">*[System generated]*</span>: the date up to which this subscription was invoiced -- for `IN_ADVANCE` billing mode, this date will often be in the future and for `IN_ARREAR`, this date will often be in the past.
* **`billingStartDate`** <span style="color:#32A9C7">*[User generated, immutable]*</span>: The (billing) start date -- date at which the system starts invoicing.
* **`billingEndDate`** <span style="color:#32A9C7">*[User generated, immutable]*</span>: The (billing) end date -- date at which the system ends invoicing.
* **`billCycleDayLocal`** <span style="color:#32A9C7">*[User or system generated]*</span>: For billing period that are multiple of months -- e.g `MONHTLY` -- this represents the date during the month at which the system triggers invoicing. The value returned is either the overridden subscription billCycleDay (in case of BCD change) or the value at the subcription, bundle or account level (depending on the catalog [billing alignments](http://docs.killbill.io/latest/userguide_subscription.html#_billing_alignment_rules)). For `ACCOUNT` billing alignments, if the account level billCycleDay hasn't been set yet, the value returned would be null.
* **`events`** <span style="color:#32A9C7">*[System generated]*</span>: List of subscription events tracking what happened -- e.g `CHANGE_PLAN`.
* **`price`** <span style="color:#32A9C7">*[User generated]*</span>: A list of prices, one for each phase associated with the `Plan` matching this subscription.
* **`priceOverrides`** <span style="color:#32A9C7">*[User generated]*</span>: This is similar to the previous field `price`, but is only filled when the prices attached to the `Plan` were overriden for this specific subscription (price override feature).


## Subscriptions

### Create a subscription

**HTTP Request** 

`POST http://127.0.0.1:8080/1.0/kb/subscriptions`

> Example Request:

```shell
curl -v \
    -X POST \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Content-Type: application/json" \
    -H "X-Killbill-CreatedBy: demo" \
    -d '{ 
            "accountId": "581d86fc-7cfc-46f2-b6d4-4dbc1d98beb3",
            "externalKey": "somethingSpecial",
            "planName": "super-monthly"
        }' \
    "http://127.0.0.1:8080/1.0/kb/subscriptions" 
```

```java
import org.killbill.billing.client.api.gen.SubscriptionApi;
protected SubscriptionApi subscriptionApi;

UUID accountId = UUID.fromString("905a0636-ab63-40c0-acd4-b461b6808b5d");

Subscription input = new Subscription();
input.setAccountId(accountJson.getAccountId());
input.setExternalKey("somethingSpecial");
input.setPlanName("shotgun-monthly");

LocalDate entitlementDate = null;
LocalDate billingDate = null;
Boolean renameKeyIfExistsAndUnused = null; 
Boolean migrated = null;
Integer bcd = null;
Boolean callCompletion = true;
long DEFAULT_WAIT_COMPLETION_TIMEOUT_SEC = 10;
ImmutableMap<String, String> NULL_PLUGIN_PROPERTIES = null;

Subscription subscription = subscriptionApi.createSubscription(input, 
                                                               entitlementDate, 
                                                               billingDate, 
                                                               renameKeyIfExistsAndUnused, 
                                                               migrated, 
                                                               bcd, 
                                                               callCompletion, 
                                                               DEFAULT_WAIT_COMPLETION_TIMEOUT_SEC, 
                                                               NULL_PLUGIN_PROPERTIES, 
                                                               requestOptions);
```

```ruby
subscription              = KillBillClient::Model::Subscription.new
subscription.account_id   = "e1826665-4524-4d57-81b5-a5eb11146f3f"
subscription.plan_name    = "basic-monthly-in-advance"

requested_date  = nil
call_completion = nil 

subscription.create(user, 
                    reason, 
                    comment, 
                    requested_date, 
                    call_completion, 
                    options)
```

```python
subscriptionApi = killbill.api.SubscriptionApi()
account_id = 'e1826665-4524-4d57-81b5-a5eb11146f3f'
body = Subscription(account_id=account_id, plan_name='standard-monthly')

subscriptionApi.create_subscription(body,
                                    created_by,
                                    api_key,
                                    api_secret)
```

> Example Response:

```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 201 Created
< Location: http://127.0.0.1:8080/1.0/kb/subscriptions/77e23878-8b9d-403b-bf31-93003e125712
< Content-Type: application/json
< Content-Length: 0
```

```java
class Subscription {
    org.killbill.billing.client.model.gen.Subscription@49563466
    accountId: e8cd6795-0da1-4848-831f-51977eed42b1
    bundleId: eac6eecf-2060-434a-b472-170f80a7591c
    subscriptionId: a74081ee-d7bb-4387-a1df-34e962e37699
    externalKey: somethingSpecial
    bundleExternalKey: somethingAlsoSpecial
    startDate: 2012-04-25
    productName: Shotgun
    productCategory: BASE
    billingPeriod: MONTHLY
    phaseType: TRIAL
    priceList: DEFAULT
    planName: shotgun-monthly
    state: ACTIVE
    sourceType: NATIVE
    cancelledDate: null
    chargedThroughDate: null
    billingStartDate: 2012-04-25
    billingEndDate: null
    billCycleDayLocal: 25
    events: [class EventSubscription {
        org.killbill.billing.client.model.gen.EventSubscription@37b70727
        eventId: 9ef798a3-95f6-41ac-9b86-00c9385c155f
        billingPeriod: MONTHLY
        effectiveDate: 2012-04-25
        plan: shotgun-monthly
        product: Shotgun
        priceList: DEFAULT
        eventType: START_ENTITLEMENT
        isBlockedBilling: false
        isBlockedEntitlement: false
        serviceName: entitlement-service
        serviceStateName: ENT_STARTED
        phase: shotgun-monthly-trial
        auditLogs: []
    }, class EventSubscription {
        org.killbill.billing.client.model.gen.EventSubscription@410923f5
        eventId: 65ec07fa-61c7-4f05-bd6f-82cea23cf06a
        billingPeriod: MONTHLY
        effectiveDate: 2012-04-25
        plan: shotgun-monthly
        product: Shotgun
        priceList: DEFAULT
        eventType: START_BILLING
        isBlockedBilling: false
        isBlockedEntitlement: false
        serviceName: billing-service
        serviceStateName: START_BILLING
        phase: shotgun-monthly-trial
        auditLogs: []
    }, class EventSubscription {
        org.killbill.billing.client.model.gen.EventSubscription@cac84db3
        eventId: 88f77a50-edca-4cc3-b234-5d70c457128c
        billingPeriod: MONTHLY
        effectiveDate: 2012-05-25
        plan: shotgun-monthly
        product: Shotgun
        priceList: DEFAULT
        eventType: PHASE
        isBlockedBilling: false
        isBlockedEntitlement: false
        serviceName: entitlement+billing-service
        serviceStateName: PHASE
        phase: shotgun-monthly-evergreen
        auditLogs: []
    }]
    priceOverrides: null
    prices: [class PhasePrice {
        planName: shotgun-monthly
        phaseName: shotgun-monthly-trial
        phaseType: TRIAL
        fixedPrice: 0
        recurringPrice: null
        usagePrices: []
    }, class PhasePrice {
        planName: shotgun-monthly
        phaseName: shotgun-monthly-evergreen
        phaseType: EVERGREEN
        fixedPrice: null
        recurringPrice: 249.95
        usagePrices: []
    }]
    auditLogs: []
}
```
```ruby
{
   "accountId":"e1826665-4524-4d57-81b5-a5eb11146f3f",
   "bundleId":"f3dea847-1567-467a-8373-838dfdcf6afc",
   "subscriptionId":"ee508b5b-46b8-42a7-8988-16c0470de4ae",
   "externalKey":"f3dea847-1567-467a-8373-838dfdcf6afc",
   "bundleExternalKey":"addea847-1467-167a-1373-988dfdcf7acc",
   "startDate":"2013-08-01",
   "productName":"Basic",
   "productCategory":"BASE",
   "billingPeriod":"MONTHLY",
   "phaseType":"EVERGREEN",
   "priceList":"DEFAULT",
   "planName":"basic-monthly-in-advance",
   "state":"ACTIVE",
   "sourceType":"NATIVE",
   "chargedThroughDate":"2013-09-01",
   "billingStartDate":"2013-08-01",
   "billCycleDayLocal":1,
   "events":[
      {
         "eventId":"341fc529-612b-4bb9-b8d7-ee4a9115f577",
         "billingPeriod":"MONTHLY",
         "effectiveDate":"2013-08-01",
         "plan":"basic-monthly-in-advance",
         "product":"Basic",
         "priceList":"DEFAULT",
         "eventType":"START_ENTITLEMENT",
         "isBlockedBilling":false,
         "isBlockedEntitlement":false,
         "serviceName":"entitlement-service",
         "serviceStateName":"ENT_STARTED",
         "phase":"basic-monthly-in-advance-evergreen",
         "auditLogs":[]
      },
      {
         "eventId":"caa54161-c001-44a0-9ff0-80be59989380",
         "billingPeriod":"MONTHLY",
         "effectiveDate":"2013-08-01",
         "plan":"basic-monthly-in-advance",
         "product":"Basic",
         "priceList":"DEFAULT",
         "eventType":"START_BILLING",
         "isBlockedBilling":false,
         "isBlockedEntitlement":false,
         "serviceName":"billing-service",
         "serviceStateName":"START_BILLING",
         "phase":"basic-monthly-in-advance-evergreen",
         "auditLogs":[]
      }
   ],
   "prices":[
      {
         "planName":"basic-monthly-in-advance",
         "phaseName":"asic-monthly-in-advance-trial",
         "phaseType":"TRIAL",
         "fixedPrice":0,
         "recurringPrice":null,
         "usagePrices":[]
      },
      {
         "planName":"basic-monthly-in-advance",
         "phaseName":"basic-monthly-in-advance-evergreen",
         "phaseType":"EVERGREEN",
         "fixedPrice":null,
         "recurringPrice":500.0,
         "usagePrices":[]
      }
   ],
   "auditLogs":[]
}
```
```python
no content
```

**Query Parameters**

| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| **entitlementDate** | string | false | the date at which the entitlement (sevice) starts. A null date means right now. |
| **billingDate** | string | false | the date at which the billing starts A null date means right now. |
| **renameKeyIfExistsAndUnused** | boolean | false | rename external key if exists and unused (default: true) |
| **migrated** | boolean | false | choose true if is migrated (default: false) |
| **bcd** | integer | false | Override the bill cycle day for this subscription. |
| **callCompletion** | boolean | false | ability to block the call until invoice and payment, if any, have been generated. (default: false)|
| **callTimeoutSec** | long | false | when setting callCompletion=true, timeout in sec |

When creating an add-on subscription, you need to specify the associated bundle in the body, either via its id or via its external key (but not both).

**Returns**

Returns a subscription object.

### Create a subscription with addOn products

**HTTP Request**

`POST http://127.0.0.1:8080/1.0/kb/subscriptions/createSubscriptionWithAddOns`

> Example Request:

```shell
curl -v \
    -X POST \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Content-Type: application/json" \
    -H "X-Killbill-CreatedBy: demo" \
    -d '[
          {
            "accountId": "581d86fc-7cfc-46f2-b6d4-4dbc1d98beb3",
            "externalKey": "something",
            "bundleExternalKey": "somethingAlso",
            "productName": "Sports",
            "productCategory": "BASE",
            "billingPeriod": "MONTHLY",
            "priceList": "DEFAULT"
        },
        {
          "accountId": "581d86fc-7cfc-46f2-b6d4-4dbc1d98beb3",
          "productName": "OilSlick",
          "productCategory": "ADD_ON",
          "billingPeriod": "MONTHLY",
          "priceList": "DEFAULT"
        }
        ]' \
    'http://127.0.0.1:8080/1.0/kb/subscriptions/createSubscriptionWithAddOns' 
```

```java
import org.killbill.billing.client.api.gen.SubscriptionApi;
protected SubscriptionApi subscriptionApi;

UUID accountId = UUID.fromString("a3087bfb-eb81-466d-afeb-6501c30f8f85");

Subscription base = new Subscription();
base.setAccountId(accountId);
base.setExternalKey("base");
base.setProductName("Shotgun");
base.setProductCategory(ProductCategory.BASE);
base.setBillingPeriod(BillingPeriod.MONTHLY);
base.setPriceList("DEFAULT");

final Subscription addOn1 = new Subscription();
addOn1.setAccountId(accountId);
addOn1.setProductName("Telescopic-Scope");
addOn1.setProductCategory(ProductCategory.ADD_ON);
addOn1.setBillingPeriod(BillingPeriod.MONTHLY);
addOn1.setPriceList("DEFAULT");

final Subscription addOn2 = new Subscription();
addOn2.setAccountId(accountId);
addOn2.setProductName("Laser-Scope");
addOn2.setProductCategory(ProductCategory.ADD_ON);
addOn2.setBillingPeriod(BillingPeriod.MONTHLY);
addOn2.setPriceList("DEFAULT");

final Subscriptions subscriptions = new Subscriptions();
subscriptions.add(base);
subscriptions.add(addOn1);
subscriptions.add(addOn2);

BulkSubscriptionsBundles bulkSubscriptionsBundles = new BulkSubscriptionsBundles();

BulkSubscriptionsBundle bulkSubscriptionsBundle = new BulkSubscriptionsBundle();
bulkSubscriptionsBundle.setBaseEntitlementAndAddOns(subscriptions);
bulkSubscriptionsBundles.add(bulkSubscriptionsBundle);

LocalDate entitlementDate = null;
LocalDate billingDate = null;
Boolean renameKeyIfExistsAndUnused = false;
Boolean migrated = false;
Boolean callCompletion = true;
long DEFAULT_WAIT_COMPLETION_TIMEOUT_SEC = 10;
ImmutableMap<String, String> NULL_PLUGIN_PROPERTIES = null;

Bundles bundles = subscriptionApi.createSubscriptionsWithAddOns(bulkSubscriptionsBundles, 
                                                                entitlementDate, 
                                                                billingDate, 
                                                                renameKeyIfExistsAndUnused, 
                                                                migrated, 
                                                                callCompletion, 
                                                                DEFAULT_WAIT_COMPLETION_TIMEOUT_SEC,
                                                                NULL_PLUGIN_PROPERTIES, 
                                                                requestOptions);
```

```ruby
entitlement = [
                 {
                    "baseEntitlementAndAddOns":[
                       {
                          "accountId":"16cd9eb8-bb5d-4183-b8e0-c1d6f78dc836",
                          "externalKey":"1-16cd9eb8-bb5d-4183-b8e0-c1d6f78dc836-827963",
                          "productCategory":"BASE",
                          "planName":"sports-monthly"
                       }
                    ]
                 }
              ]
requested_date = nil
entitlement_date = nil
billing_date = nil
migrated = false
call_completion_sec = 3

subscription = KillBillClient::Model::Subscription.new
subscription.create_entitlement_with_add_on(entitlement,
                                            requested_date,
                                            entitlement_date, 
                                            billing_date,
                                            migrated, 
                                            call_completion_sec,                                            
                                            user, 
                                            reason, 
                                            comment, 
                                            options)                                         
```

```python
subscriptionApi = killbill.api.SubscriptionApi()
account_id = '16cd9eb8-bb5d-4183-b8e0-c1d6f78dc836'
subscription_a = Subscription(account_id=account_id,
                              product_category='BASE',
                              plan_name='sports-monthly')

subscription_b = Subscription(account_id=account_id,
                              product_category='ADD_ON',
                              plan_name='super-monthly')

body = [subscription_a, subscription_b]

subscriptionApi.create_subscription_with_add_ons(body,
                                                 created_by,
                                                 api_key,
                                                 api_secret)
```

> Example Response:

```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 201 Created
< Location: http://127.0.0.1:8080/1.0/kb/accounts/886adb60-be70-40c8-b97d-1f8ecbc30a64/bundles?bundlesFilter=cbcc7642-1aa5-4609-b89d-5356d05819be
< Content-Type: application/json
< Content-Length: 0
```

```java
class BulkSubscriptionsBundle {
    baseEntitlementAndAddOns: [class Subscription {
        org.killbill.billing.client.model.gen.Subscription@23b30120
        accountId: a3087bfb-eb81-466d-afeb-6501c30f8f85
        bundleId: 2df85b05-4a69-474b-9e10-02a836674cc8
        subscriptionId: null
        externalKey: null
        startDate: null
        productName: Telescopic-Scope
        productCategory: ADD_ON
        billingPeriod: MONTHLY
        phaseType: null
        priceList: DEFAULT
        planName: null
        state: null
        sourceType: null
        cancelledDate: null
        chargedThroughDate: null
        billingStartDate: null
        billingEndDate: null
        billCycleDayLocal: null
        events: null
        priceOverrides: null
        prices: null
        auditLogs: null
    }, class Subscription {
        org.killbill.billing.client.model.gen.Subscription@1b88c7d0
        accountId: a3087bfb-eb81-466d-afeb-6501c30f8f85
        bundleId: 2df85b05-4a69-474b-9e10-02a836674cc8
        subscriptionId: null
        externalKey: null
        startDate: null
        productName: Laser-Scope
        productCategory: ADD_ON
        billingPeriod: MONTHLY
        phaseType: null
        priceList: DEFAULT
        planName: null
        state: null
        sourceType: null
        cancelledDate: null
        chargedThroughDate: null
        billingStartDate: null
        billingEndDate: null
        billCycleDayLocal: null
        events: null
        priceOverrides: null
        prices: null
        auditLogs: null
    }]
}
```
```ruby
no content
```
```python
no content
```

**Query Parameters**

| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| **entitlementDate** | string | false | the date at which the entitlement (sevice) starts. A null date means right now.|
| **billingDate** | string | false | the date at which the billing starts A null date means right now. |
| **renameKeyIfExistsAndUnused** | boolean | false | rename key if exists and unused (default: true) |
| **migrated** | boolean | false | choose true if is migrated (default: false) |
| **callCompletion** | boolean | false | ability to block the call until invoice and payment, if any, have been generated. (default: false)|
| **callTimeoutSec** | long | false | when setting callCompletion=true, timeout in sec |

**Returns**

A `201` http status without content.


### Create multiple subscriptions with addOn products

**HTTP Request** 

`POST http://127.0.0.1:8080/1.0/kb/subscriptions/createSubscriptionsWithAddOns`

> Example Request:

```shell
curl -v \
    -X POST \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Content-Type: application/json" \
    -H "X-Killbill-CreatedBy: demo" \
    -d '[
          {
            "baseEntitlementAndAddOns": [
              {
                "accountId": "886adb60-be70-40c8-b97d-1f8ecbc30a64",
                "externalKey": "base",
                "productName": "Sports",
                "productCategory": "BASE",
                "billingPeriod": "MONTHLY",
                "priceList": "DEFAULT"
              },
              {
                "accountId": "886adb60-be70-40c8-b97d-1f8ecbc30a64",
                "productName": "OilSlick",
                "productCategory": "ADD_ON",
                "billingPeriod": "MONTHLY",
                "priceList": "DEFAULT"
              }
            ]
          }
        ]' \
    'http://127.0.0.1:8080/1.0/kb/subscriptions/createSubscriptionsWithAddOns' 
```

```java
import org.killbill.billing.client.api.gen.SubscriptionApi;
protected SubscriptionApi subscriptionApi;

UUID accountId = UUID.fromString("abfba40d-d2dd-47f0-94f2-8ea3e6bc8794");

Subscription base = new Subscription();
base.setAccountId(accountId);
base.setExternalKey("base");
base.setProductName("Shotgun");
base.setProductCategory(ProductCategory.BASE);
base.setBillingPeriod(BillingPeriod.MONTHLY);
base.setPriceList("DEFAULT");

final Subscription addOn1 = new Subscription();
addOn1.setAccountId(accountId);
addOn1.setProductName("Telescopic-Scope");
addOn1.setProductCategory(ProductCategory.ADD_ON);
addOn1.setBillingPeriod(BillingPeriod.MONTHLY);
addOn1.setPriceList("DEFAULT");

final Subscription addOn2 = new Subscription();
addOn2.setAccountId(accountId);
addOn2.setProductName("Laser-Scope");
addOn2.setProductCategory(ProductCategory.ADD_ON);
addOn2.setBillingPeriod(BillingPeriod.MONTHLY);
addOn2.setPriceList("DEFAULT");

final Subscriptions subscriptions = new Subscriptions();
subscriptions.add(base);
subscriptions.add(addOn1);
subscriptions.add(addOn2);

LocalDate entitlementDate = null;
LocalDate billingDate = null;
Boolean migrated = null;
Boolean renameKeyIfExistsAndUnused = null;
Boolean callCompletion = true;
long DEFAULT_WAIT_COMPLETION_TIMEOUT_SEC = 10;
ImmutableMap<String, String> NULL_PLUGIN_PROPERTIES = null;

final Bundle bundle = subscriptionApi.createSubscriptionWithAddOns(subscriptions, 
                                                                   entitlementDate, 
                                                                   billingDate, 
                                                                   migrated, 
                                                                   renameKeyIfExistsAndUnused, 
                                                                   callCompletion, 
                                                                   DEFAULT_WAIT_COMPLETION_TIMEOUT_SEC, 
                                                                   NULL_PLUGIN_PROPERTIES, 
                                                                   requestOptions);
```

```ruby
bulk_subscription_list = [
                            {
                               "baseEntitlementAndAddOns":[
                                  {
                                     "accountId":"16cd9eb8-bb5d-4183-b8e0-c1d6f78dc836",
                                     "externalKey":"1-16cd9eb8-bb5d-4183-b8e0-c1d6f78dc836-827963",
                                     "productCategory":"BASE",
                                     "planName":"sports-monthly"
                                  }
                               ]
                            },
                            {
                               "baseEntitlementAndAddOns":[
                                  {
                                     "accountId":"16cd9eb8-bb5d-4183-b8e0-c1d6f78dc836",
                                     "externalKey":"2-16cd9eb8-bb5d-4183-b8e0-c1d6f78dc836-717751",
                                     "productCategory":"ADD_ON",
                                     "planName":"super-monthly"
                                  }
                               ]
                            }
                         ]
entitlement_date = nil
billing_date = nil
call_completion_sec = nil

KillBillClient::Model::BulkSubscription.create_bulk_subscriptions(bulk_subscription_list, 
                                                                  user, 
                                                                  reason, 
                                                                  comment, 
                                                                  entitlement_date, 
                                                                  billing_date, 
                                                                  call_completion_sec, 
                                                                  options)
```

```python
subscriptionApi = killbill.api.SubscriptionApi()
account_id = '16cd9eb8-bb5d-4183-b8e0-c1d6f78dc836'
subscription_a = Subscription(account_id=account_id,
                              product_category='BASE',
                              plan_name='sports-monthly')

subscription_b = Subscription(account_id=account_id,
                              product_category='ADD_ON',
                              plan_name='super-monthly')

body = BulkSubscriptionsBundle([subscription_a, subscription_b])

subscriptionApi.create_subscriptions_with_add_ons([body],
                                                  created_by,
                                                  api_key,
                                                  api_secret)
```

> Example Response:

```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 201 Created
< Location: http://127.0.0.1:8080/1.0/kb/accounts/886adb60-be70-40c8-b97d-1f8ecbc30a64/bundles?bundlesFilter=cbcc7642-1aa5-4609-b89d-5356d05819be
< Content-Type: application/json
< Content-Length: 0
```

```java
class Bundle {
    org.killbill.billing.client.model.gen.Bundle@ddd846f9
    accountId: abfba40d-d2dd-47f0-94f2-8ea3e6bc8794
    bundleId: 83f3a7b5-63c9-4b06-a1f5-7140fec67558
    externalKey: base
    subscriptions: [class Subscription {
        org.killbill.billing.client.model.gen.Subscription@bb1bf160
        accountId: abfba40d-d2dd-47f0-94f2-8ea3e6bc8794
        bundleId: 83f3a7b5-63c9-4b06-a1f5-7140fec67558
        subscriptionId: 6fcf439b-4198-4963-8f0e-7e3054525a24
        externalKey: base
        startDate: 2012-04-25
        productName: Shotgun
        productCategory: BASE
        billingPeriod: MONTHLY
        phaseType: TRIAL
        priceList: DEFAULT
        planName: shotgun-monthly
        state: ACTIVE
        sourceType: NATIVE
        cancelledDate: null
        chargedThroughDate: 2012-04-25
        billingStartDate: 2012-04-25
        billingEndDate: null
        billCycleDayLocal: 25
        events: [class EventSubscription {
            org.killbill.billing.client.model.gen.EventSubscription@a6789138
            eventId: fb7d653a-cadb-46f6-a04f-dbe7043385ed
            billingPeriod: MONTHLY
            effectiveDate: 2012-04-25
            plan: shotgun-monthly
            product: Shotgun
            priceList: DEFAULT
            eventType: START_ENTITLEMENT
            isBlockedBilling: false
            isBlockedEntitlement: false
            serviceName: entitlement-service
            serviceStateName: ENT_STARTED
            phase: shotgun-monthly-trial
            auditLogs: []
        }, class EventSubscription {
            org.killbill.billing.client.model.gen.EventSubscription@b4b456ed
            eventId: b5c00fdf-6ce4-46ad-b996-5513146d8acd
            billingPeriod: MONTHLY
            effectiveDate: 2012-04-25
            plan: shotgun-monthly
            product: Shotgun
            priceList: DEFAULT
            eventType: START_BILLING
            isBlockedBilling: false
            isBlockedEntitlement: false
            serviceName: billing-service
            serviceStateName: START_BILLING
            phase: shotgun-monthly-trial
            auditLogs: []
        }, class EventSubscription {
            org.killbill.billing.client.model.gen.EventSubscription@fb1cc392
            eventId: edd91628-323d-4a3a-b07e-bef09a2623a2
            billingPeriod: MONTHLY
            effectiveDate: 2012-05-25
            plan: shotgun-monthly
            product: Shotgun
            priceList: DEFAULT
            eventType: PHASE
            isBlockedBilling: false
            isBlockedEntitlement: false
            serviceName: entitlement+billing-service
            serviceStateName: PHASE
            phase: shotgun-monthly-evergreen
            auditLogs: []
        }]
        priceOverrides: null
        prices: [class PhasePrice {
            planName: shotgun-monthly
            phaseName: shotgun-monthly-trial
            phaseType: TRIAL
            fixedPrice: 0
            recurringPrice: null
            usagePrices: []
        }, class PhasePrice {
            planName: shotgun-monthly
            phaseName: shotgun-monthly-evergreen
            phaseType: EVERGREEN
            fixedPrice: null
            recurringPrice: 249.95
            usagePrices: []
        }]
        auditLogs: []
    }, class Subscription {
        org.killbill.billing.client.model.gen.Subscription@37a59f88
        accountId: abfba40d-d2dd-47f0-94f2-8ea3e6bc8794
        bundleId: 83f3a7b5-63c9-4b06-a1f5-7140fec67558
        subscriptionId: a490a1c8-ca78-4cbd-bb5a-1ce214cc2f23
        externalKey: base
        startDate: 2012-04-25
        productName: Telescopic-Scope
        productCategory: ADD_ON
        billingPeriod: MONTHLY
        phaseType: DISCOUNT
        priceList: DEFAULT
        planName: telescopic-scope-monthly
        state: ACTIVE
        sourceType: NATIVE
        cancelledDate: null
        chargedThroughDate: 2012-05-25
        billingStartDate: 2012-04-25
        billingEndDate: null
        billCycleDayLocal: 25
        events: [class EventSubscription {
            org.killbill.billing.client.model.gen.EventSubscription@9c41861a
            eventId: 6be47e8c-0ff6-45ee-9b79-7a91c6867e6c
            billingPeriod: MONTHLY
            effectiveDate: 2012-04-25
            plan: telescopic-scope-monthly
            product: Telescopic-Scope
            priceList: DEFAULT
            eventType: START_ENTITLEMENT
            isBlockedBilling: false
            isBlockedEntitlement: false
            serviceName: entitlement-service
            serviceStateName: ENT_STARTED
            phase: telescopic-scope-monthly-discount
            auditLogs: []
        }, class EventSubscription {
            org.killbill.billing.client.model.gen.EventSubscription@a386d43d
            eventId: 0c6b12cb-9390-4d64-b98a-93a8c9a6b67b
            billingPeriod: MONTHLY
            effectiveDate: 2012-04-25
            plan: telescopic-scope-monthly
            product: Telescopic-Scope
            priceList: DEFAULT
            eventType: START_BILLING
            isBlockedBilling: false
            isBlockedEntitlement: false
            serviceName: billing-service
            serviceStateName: START_BILLING
            phase: telescopic-scope-monthly-discount
            auditLogs: []
        }, class EventSubscription {
            org.killbill.billing.client.model.gen.EventSubscription@429a9148
            eventId: 27ce7282-c84c-495a-a7fc-be393cfcf3a7
            billingPeriod: MONTHLY
            effectiveDate: 2012-05-25
            plan: telescopic-scope-monthly
            product: Telescopic-Scope
            priceList: DEFAULT
            eventType: PHASE
            isBlockedBilling: false
            isBlockedEntitlement: false
            serviceName: entitlement+billing-service
            serviceStateName: PHASE
            phase: telescopic-scope-monthly-evergreen
            auditLogs: []
        }]
        priceOverrides: null
        prices: [class PhasePrice {
            planName: telescopic-scope-monthly
            phaseName: telescopic-scope-monthly-trial
            phaseType: TRIAL
            fixedPrice: 0
            recurringPrice: null
            usagePrices: []
        }, class PhasePrice {
            planName: telescopic-scope-monthly
            phaseName: telescopic-scope-monthly-evergreen
            phaseType: EVERGREEN
            fixedPrice: null
            recurringPrice: 249.95
            usagePrices: []
        }]
        auditLogs: []
    }, class Subscription {
        org.killbill.billing.client.model.gen.Subscription@10da5c43
        accountId: abfba40d-d2dd-47f0-94f2-8ea3e6bc8794
        bundleId: 83f3a7b5-63c9-4b06-a1f5-7140fec67558
        subscriptionId: 8f957bfa-61da-45ca-ae4c-ed34b045f18e
        externalKey: base
        startDate: 2012-04-25
        productName: Laser-Scope
        productCategory: ADD_ON
        billingPeriod: MONTHLY
        phaseType: DISCOUNT
        priceList: DEFAULT
        planName: laser-scope-monthly
        state: ACTIVE
        sourceType: NATIVE
        cancelledDate: null
        chargedThroughDate: 2012-05-25
        billingStartDate: 2012-04-25
        billingEndDate: null
        billCycleDayLocal: 25
        events: [class EventSubscription {
            org.killbill.billing.client.model.gen.EventSubscription@6615c622
            eventId: eb05b9d5-a055-426c-8411-7fd9b4728d45
            billingPeriod: MONTHLY
            effectiveDate: 2012-04-25
            plan: laser-scope-monthly
            product: Laser-Scope
            priceList: DEFAULT
            eventType: START_ENTITLEMENT
            isBlockedBilling: false
            isBlockedEntitlement: false
            serviceName: entitlement-service
            serviceStateName: ENT_STARTED
            phase: laser-scope-monthly-discount
            auditLogs: []
        }, class EventSubscription {
            org.killbill.billing.client.model.gen.EventSubscription@41ddb948
            eventId: 6fbe3e71-7e3b-4c3b-8bdd-27c8dc71e044
            billingPeriod: MONTHLY
            effectiveDate: 2012-04-25
            plan: laser-scope-monthly
            product: Laser-Scope
            priceList: DEFAULT
            eventType: START_BILLING
            isBlockedBilling: false
            isBlockedEntitlement: false
            serviceName: billing-service
            serviceStateName: START_BILLING
            phase: laser-scope-monthly-discount
            auditLogs: []
        }, class EventSubscription {
            org.killbill.billing.client.model.gen.EventSubscription@182c6fd0
            eventId: bec40e45-6ffb-49f3-831d-722f19d8ed09
            billingPeriod: MONTHLY
            effectiveDate: 2012-05-25
            plan: laser-scope-monthly
            product: Laser-Scope
            priceList: DEFAULT
            eventType: PHASE
            isBlockedBilling: false
            isBlockedEntitlement: false
            serviceName: entitlement+billing-service
            serviceStateName: PHASE
            phase: laser-scope-monthly-evergreen
            auditLogs: []
        }]
        priceOverrides: null
        prices: [class PhasePrice {
            planName: laser-scope-monthly
            phaseName: laser-scope-monthly-trial
            phaseType: TRIAL
            fixedPrice: 0
            recurringPrice: null
            usagePrices: []
        }, class PhasePrice {
            planName: laser-scope-monthly
            phaseName: laser-scope-monthly-evergreen
            phaseType: EVERGREEN
            fixedPrice: null
            recurringPrice: 249.95
            usagePrices: []
        }]
        auditLogs: []
    }]
    timeline: class BundleTimeline {
        org.killbill.billing.client.model.gen.BundleTimeline@8a202296
        accountId: abfba40d-d2dd-47f0-94f2-8ea3e6bc8794
        bundleId: 83f3a7b5-63c9-4b06-a1f5-7140fec67558
        externalKey: base
        events: [class EventSubscription {
            org.killbill.billing.client.model.gen.EventSubscription@5aa4f7e9
            eventId: eb05b9d5-a055-426c-8411-7fd9b4728d45
            billingPeriod: MONTHLY
            effectiveDate: 2012-04-25
            plan: laser-scope-monthly
            product: Laser-Scope
            priceList: DEFAULT
            eventType: START_ENTITLEMENT
            isBlockedBilling: false
            isBlockedEntitlement: false
            serviceName: entitlement-service
            serviceStateName: ENT_STARTED
            phase: laser-scope-monthly-discount
            auditLogs: []
        }, class EventSubscription {
            org.killbill.billing.client.model.gen.EventSubscription@1df149f3
            eventId: 6be47e8c-0ff6-45ee-9b79-7a91c6867e6c
            billingPeriod: MONTHLY
            effectiveDate: 2012-04-25
            plan: laser-scope-monthly
            product: Laser-Scope
            priceList: DEFAULT
            eventType: START_ENTITLEMENT
            isBlockedBilling: false
            isBlockedEntitlement: false
            serviceName: entitlement-service
            serviceStateName: ENT_STARTED
            phase: laser-scope-monthly-discount
            auditLogs: []
        }, class EventSubscription {
            org.killbill.billing.client.model.gen.EventSubscription@68a2bc3b
            eventId: fb7d653a-cadb-46f6-a04f-dbe7043385ed
            billingPeriod: MONTHLY
            effectiveDate: 2012-04-25
            plan: laser-scope-monthly
            product: Laser-Scope
            priceList: DEFAULT
            eventType: START_ENTITLEMENT
            isBlockedBilling: false
            isBlockedEntitlement: false
            serviceName: entitlement-service
            serviceStateName: ENT_STARTED
            phase: laser-scope-monthly-discount
            auditLogs: []
        }, class EventSubscription {
            org.killbill.billing.client.model.gen.EventSubscription@787b414d
            eventId: 6fbe3e71-7e3b-4c3b-8bdd-27c8dc71e044
            billingPeriod: MONTHLY
            effectiveDate: 2012-04-25
            plan: laser-scope-monthly
            product: Laser-Scope
            priceList: DEFAULT
            eventType: START_BILLING
            isBlockedBilling: false
            isBlockedEntitlement: false
            serviceName: billing-service
            serviceStateName: START_BILLING
            phase: laser-scope-monthly-discount
            auditLogs: []
        }, class EventSubscription {
            org.killbill.billing.client.model.gen.EventSubscription@5ba02249
            eventId: 0c6b12cb-9390-4d64-b98a-93a8c9a6b67b
            billingPeriod: MONTHLY
            effectiveDate: 2012-04-25
            plan: telescopic-scope-monthly
            product: Telescopic-Scope
            priceList: DEFAULT
            eventType: START_BILLING
            isBlockedBilling: false
            isBlockedEntitlement: false
            serviceName: billing-service
            serviceStateName: START_BILLING
            phase: telescopic-scope-monthly-discount
            auditLogs: []
        }, class EventSubscription {
            org.killbill.billing.client.model.gen.EventSubscription@d45c4b96
            eventId: b5c00fdf-6ce4-46ad-b996-5513146d8acd
            billingPeriod: MONTHLY
            effectiveDate: 2012-04-25
            plan: shotgun-monthly
            product: Shotgun
            priceList: DEFAULT
            eventType: START_BILLING
            isBlockedBilling: false
            isBlockedEntitlement: false
            serviceName: billing-service
            serviceStateName: START_BILLING
            phase: shotgun-monthly-trial
            auditLogs: []
        }, class EventSubscription {
            org.killbill.billing.client.model.gen.EventSubscription@2360e010
            eventId: bec40e45-6ffb-49f3-831d-722f19d8ed09
            billingPeriod: MONTHLY
            effectiveDate: 2012-05-25
            plan: laser-scope-monthly
            product: Laser-Scope
            priceList: DEFAULT
            eventType: PHASE
            isBlockedBilling: false
            isBlockedEntitlement: false
            serviceName: entitlement+billing-service
            serviceStateName: PHASE
            phase: laser-scope-monthly-evergreen
            auditLogs: []
        }, class EventSubscription {
            org.killbill.billing.client.model.gen.EventSubscription@49d428fe
            eventId: 27ce7282-c84c-495a-a7fc-be393cfcf3a7
            billingPeriod: MONTHLY
            effectiveDate: 2012-05-25
            plan: telescopic-scope-monthly
            product: Telescopic-Scope
            priceList: DEFAULT
            eventType: PHASE
            isBlockedBilling: false
            isBlockedEntitlement: false
            serviceName: entitlement+billing-service
            serviceStateName: PHASE
            phase: telescopic-scope-monthly-evergreen
            auditLogs: []
        }, class EventSubscription {
            org.killbill.billing.client.model.gen.EventSubscription@aebff2be
            eventId: edd91628-323d-4a3a-b07e-bef09a2623a2
            billingPeriod: MONTHLY
            effectiveDate: 2012-05-25
            plan: shotgun-monthly
            product: Shotgun
            priceList: DEFAULT
            eventType: PHASE
            isBlockedBilling: false
            isBlockedEntitlement: false
            serviceName: entitlement+billing-service
            serviceStateName: PHASE
            phase: shotgun-monthly-evergreen
            auditLogs: []
        }]
        auditLogs: []
    }
    auditLogs: []
}
```
```ruby
no content
```
```python
no content
```


**Query Parameters**

| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| **entitlementDate** | string | false | the date at which the entitlements (sevice) starts. A null date means right now. |
| **billingDate** | string | false | the date at which the billing starts A null date means right now. |
| **renameKeyIfExistsAndUnused** | boolean | false | Rename key if exists and unused (default: true) |
| **migrated** | boolean | false | choose true if is migrated (default: false) |
| **callCompletion** | boolean | false | ability to block the call until invoice and payment, if any, have been generated. (default: false)|
| **callTimeoutSec** | long | false | when setting callCompletion=true, timeout in sec |

**Returns**

A `201` http status without content.


### Retrieve a subscription by id

**HTTP Request** 

`GET http://127.0.0.1:8080/1.0/kb/subscriptions/{subscriptionId}`

> Example Request:

```shell
curl \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Accept: application/json" \
    'http://127.0.0.1:8080/1.0/kb/subscriptions/d4a919f4-7459-494f-85e5-af8880f63e90' 
```

```java
import org.killbill.billing.client.api.gen.SubscriptionApi;
protected SubscriptionApi subscriptionApi;

UUID subscriptionId = UUID.fromString("905a0636-ab63-40c0-acd4-b461b6808b5d");

Subscription objFromJson = subscriptionApi.getSubscription(subscriptionId, 
                                                           AuditLevel.NONE, 
                                                           requestOptions);
```

```ruby
subscription_id = "161692a4-c293-410c-a92f-939c5e3dcba7"

KillBillClient::Model::Subscription.find_by_id(subscription_id, options)
```

```python
subscriptionApi = killbill.api.SubscriptionApi()
subscription_id = '4aab9b96-c2e7-4641-a6d9-db984969201e'

subscriptionApi.get_subscription(subscription_id, api_key, api_secret)
```

> Example Response:

```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 200 OK
< Content-Type: application/json
<
{
    "accountId":"581d86fc-7cfc-46f2-b6d4-4dbc1d98beb3",
    "bundleId":"3b7a754c-4fe3-49a4-a56c-c8f56fc4116c",
    "subscriptionId":"d4a919f4-7459-494f-85e5-af8880f63e90",
    "externalKey":"somethingSpecial",
    "bundleExternalKey":"somethingAlsoSpecial",
    "startDate":"2018-07-19",
    "productName":"Super",
    "productCategory":"BASE",
    "billingPeriod":"MONTHLY",
    "phaseType":"TRIAL",
    "priceList":"DEFAULT",
    "planName":"super-monthly",
    "state":"ACTIVE",
    "sourceType":"NATIVE",
    "cancelledDate":null,
    "chargedThroughDate":"2018-07-19",
    "billingStartDate":"2018-07-19",
    "billingEndDate":null,
    "billCycleDayLocal":18,
    "events":
    [
        {"
            eventId":"d1fc7c9a-bdcd-447c-90f4-72c8de37d457",
            "billingPeriod":"MONTHLY",
            "effectiveDate":"2018-07-19",
            "plan":"super-monthly",
            "product":"Super",
            "priceList":"DEFAULT",
            "eventType":"START_ENTITLEMENT",
            "isBlockedBilling":false,
            "isBlockedEntitlement":false,
            "serviceName":"entitlement-service",
            "serviceStateName":"ENT_STARTED",
            "phase":"super-monthly-trial",
            "auditLogs":[]
        },
        {
            "eventId":"e1cea834-9c21-450a-8ff5-8e1ebef705d1",
            "billingPeriod":"MONTHLY",
            "effectiveDate":"2018-07-19",
            "plan":"super-monthly",
            "product":"Super",
            "priceList":"DEFAULT",
            "eventType":"START_BILLING",
            "isBlockedBilling":false,
            "isBlockedEntitlement":false,
            "serviceName":"billing-service",
            "serviceStateName":"START_BILLING",
            "phase":"super-monthly-trial",
            "auditLogs":[]
        },
        {
            "eventId":"c9045227-4638-46ca-9a4a-2d3086168505",
            "billingPeriod":"MONTHLY",
            "effectiveDate":"2018-08-18",
            "plan":"super-monthly",
            "product":"Super",
            "priceList":"DEFAULT",
            "eventType":"PHASE",
            "isBlockedBilling":false,
            "isBlockedEntitlement":false,
            "serviceName":"entitlement+billing-service",
            "serviceStateName":"PHASE",
            "phase":"super-monthly-evergreen",
            "auditLogs":[]
        }
    ],
    "priceOverrides":null,
    "prices":
    [
        {
            "planName":"super-monthly",
            "phaseName":"super-monthly-trial",
            "phaseType":"TRIAL",
            "fixedPrice":0,
            "recurringPrice":null,
            "usagePrices":[]
        },
        {
            "planName":"super-monthly",
            "phaseName":"super-monthly-evergreen",
            "phaseType":"EVERGREEN",
            "fixedPrice":null,
            "recurringPrice":1000.00,
            "usagePrices":[]
        }
    ],
    "auditLogs":[]}
```
            

```java
class Subscription {
    org.killbill.billing.client.model.gen.Subscription@c620bf78
    accountId: e903d1d6-7423-40fe-8ab7-d9d449484cda
    bundleId: 603db1e6-299e-4b8b-9dfc-beecccca39b2
    subscriptionId: b0f8f527-78c6-4fef-8fb2-53c9ed60a714
    externalKey: 99999
    bundleExternalKey: 88888
    startDate: 2012-04-25
    productName: Shotgun
    productCategory: BASE
    billingPeriod: ANNUAL
    phaseType: TRIAL
    priceList: DEFAULT
    planName: shotgun-annual
    state: ACTIVE
    sourceType: NATIVE
    cancelledDate: null
    chargedThroughDate: 2012-04-25
    billingStartDate: 2012-04-25
    billingEndDate: null
    billCycleDayLocal: 25
    events: [class EventSubscription {
        org.killbill.billing.client.model.gen.EventSubscription@c8078c46
        eventId: 7227bab2-b394-42de-b40c-97cecfd1d9ae
        billingPeriod: ANNUAL
        effectiveDate: 2012-04-25
        plan: shotgun-annual
        product: Shotgun
        priceList: DEFAULT
        eventType: START_ENTITLEMENT
        isBlockedBilling: false
        isBlockedEntitlement: false
        serviceName: entitlement-service
        serviceStateName: ENT_STARTED
        phase: shotgun-annual-trial
        auditLogs: []
    }, class EventSubscription {
        org.killbill.billing.client.model.gen.EventSubscription@1be0ca21
        eventId: ce8578e7-553f-45d7-a4d1-2abb7e625f64
        billingPeriod: ANNUAL
        effectiveDate: 2012-04-25
        plan: shotgun-annual
        product: Shotgun
        priceList: DEFAULT
        eventType: START_BILLING
        isBlockedBilling: false
        isBlockedEntitlement: false
        serviceName: billing-service
        serviceStateName: START_BILLING
        phase: shotgun-annual-trial
        auditLogs: []
    }, class EventSubscription {
        org.killbill.billing.client.model.gen.EventSubscription@f0472744
        eventId: 26d5ea05-b91d-43e6-8aa2-ac0ac82f6a95
        billingPeriod: ANNUAL
        effectiveDate: 2012-05-25
        plan: shotgun-annual
        product: Shotgun
        priceList: DEFAULT
        eventType: PHASE
        isBlockedBilling: false
        isBlockedEntitlement: false
        serviceName: entitlement+billing-service
        serviceStateName: PHASE
        phase: shotgun-annual-evergreen
        auditLogs: []
    }]
    priceOverrides: null
    prices: [class PhasePrice {
        planName: shotgun-annual
        phaseName: shotgun-annual-trial
        phaseType: TRIAL
        fixedPrice: 0
        recurringPrice: null
        usagePrices: []
    }, class PhasePrice {
        planName: shotgun-annual
        phaseName: shotgun-annual-evergreen
        phaseType: EVERGREEN
        fixedPrice: null
        recurringPrice: 249.95
        usagePrices: []
    }]
    auditLogs: []
}
```
```ruby
{
   "accountId":"0cdaeca7-4984-47dc-b245-7c32627f26cd",
   "bundleId":"d1f4ca8d-be47-4e64-84ce-f697b42d4182",
   "subscriptionId":"161692a4-c293-410c-a92f-939c5e3dcba7",
   "externalKey":"d1f4ca8d-be47-4e64-84ce-f697b42d4182",
   "bundleExternalKey":"a4f4ca8d-3447-4e64-84ce-6697b42d419c",
   "startDate":"2013-08-01",
   "productName":"Basic",
   "productCategory":"BASE",
   "billingPeriod":"MONTHLY",
   "phaseType":"EVERGREEN",
   "priceList":"DEFAULT",
   "planName":"basic-monthly-in-advance",
   "state":"ACTIVE",
   "sourceType":"NATIVE",
   "chargedThroughDate":"2013-09-01",
   "billingStartDate":"2013-08-01",
   "billCycleDayLocal":1,
   "events":[
      {
         "eventId":"dda11bf3-f74a-4c42-83e1-0f43a41389af",
         "billingPeriod":"MONTHLY",
         "effectiveDate":"2013-08-01",
         "plan":"basic-monthly-in-advance",
         "product":"Basic",
         "priceList":"DEFAULT",
         "eventType":"START_ENTITLEMENT",
         "isBlockedBilling":false,
         "isBlockedEntitlement":false,
         "serviceName":"entitlement-service",
         "serviceStateName":"ENT_STARTED",
         "phase":"basic-monthly-in-advance-evergreen",
         "auditLogs":[]
      },
      {
         "eventId":"6901117c-4ce0-4eb6-8642-380823490fae",
         "billingPeriod":"MONTHLY",
         "effectiveDate":"2013-08-01",
         "plan":"basic-monthly-in-advance",
         "product":"Basic",
         "priceList":"DEFAULT",
         "eventType":"START_BILLING",
         "isBlockedBilling":false,
         "isBlockedEntitlement":false,
         "serviceName":"billing-service",
         "serviceStateName":"START_BILLING",
         "phase":"basic-monthly-in-advance-evergreen",
         "auditLogs":[]
      }
   ],
   "prices":[
      {
         "planName":"basic-monthly-in-advance",
         "phaseName":"asic-monthly-in-advance-trial",
         "phaseType":"TRIAL",
         "fixedPrice":0,
         "recurringPrice":null,
         "usagePrices":[]
      },
      {
         "planName":"basic-monthly-in-advance",
         "phaseName":"basic-monthly-in-advance-evergreen",
         "phaseType":"EVERGREEN",
         "fixedPrice":null,
         "recurringPrice":500.0,
         "usagePrices":[]
      }
   ],
   "auditLogs":[]
}
```
```python
{'account_id': '3b1a5a67-f0ac-475c-9aad-735d309f0c1f',
 'audit_logs': [],
 'bill_cycle_day_local': 13,
 'billing_end_date': None,
 'billing_period': 'MONTHLY',
 'billing_start_date': datetime.date(2018, 5, 14),
 'bundle_id': 'e5590623-ccd4-4a8a-be26-008ce7c02b3b',
 'cancelled_date': None,
 'charged_through_date': None,
 'events': [{'audit_logs': [],
             'billing_period': 'MONTHLY',
             'effective_date': datetime.date(2018, 5, 14),
             'event_id': '46335d3d-8234-49c3-af1a-dcf8cd354ef3',
             'event_type': 'START_ENTITLEMENT',
             'is_blocked_billing': False,
             'is_blocked_entitlement': False,
             'phase': 'standard-monthly-trial',
             'plan': 'standard-monthly',
             'price_list': 'DEFAULT',
             'product': 'Standard',
             'service_name': 'entitlement-service',
             'service_state_name': 'ENT_STARTED'},
            {'audit_logs': [],
             'billing_period': 'MONTHLY',
             'effective_date': datetime.date(2018, 5, 14),
             'event_id': 'a14d6512-7479-46e0-b72b-81dff575d1d4',
             'event_type': 'START_BILLING',
             'is_blocked_billing': False,
             'is_blocked_entitlement': False,
             'phase': 'standard-monthly-trial',
             'plan': 'standard-monthly',
             'price_list': 'DEFAULT',
             'product': 'Standard',
             'service_name': 'billing-service',
             'service_state_name': 'START_BILLING'},
            {'audit_logs': [],
             'billing_period': 'MONTHLY',
             'effective_date': datetime.date(2018, 6, 13),
             'event_id': '27408c65-46b8-4bc9-a7ee-c80d6e5fb9b5',
             'event_type': 'PHASE',
             'is_blocked_billing': False,
             'is_blocked_entitlement': False,
             'phase': 'standard-monthly-evergreen',
             'plan': 'standard-monthly',
             'price_list': 'DEFAULT',
             'product': 'Standard',
             'service_name': 'entitlement+billing-service',
             'service_state_name': 'PHASE'}],
 'external_key': 'e5590623-ccd4-4a8a-be26-008ce7c02b3b',
 'phase_type': 'TRIAL',
 'plan_name': 'standard-monthly',
 'price_list': 'DEFAULT',
 'prices': [{'fixed_price': 0.0,
              'phase_name': 'standard-monthly-trial',
              'phase_type': 'TRIAL',
              'plan_name': 'standard-monthly',
              'recurring_price': None,
              'usage_prices': []},
             {'fixed_price': None,
              'phase_name': 'standard-monthly-evergreen',
              'phase_type': 'EVERGREEN',
              'plan_name': 'standard-monthly',
              'recurring_price': 100.0,
              'usage_prices': []}],  
 'product_category': 'BASE',
 'product_name': 'Standard',
 'source_type': 'NATIVE',
 'start_date': datetime.date(2018, 5, 14),
 'state': 'ACTIVE',
 'subscription_id': '4aab9b96-c2e7-4641-a6d9-db984969201e'}
```
**Query Parameters**

| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| **audit** | enum | false | level of audit logs returned |

**Returns**

Returns a subscription object.


### Retrieve a subscription by key

**HTTP Request**

`GET http://127.0.0.1:8080/1.0/kb/subscriptions/`

> Example Request:

```shell
curl -v \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Accept: application/json" \
    'http://localhost:8080/1.0/kb/subscriptions?externalKey=somethingSpecial'
```

```java
import org.killbill.billing.client.api.gen.SubscriptionApi;
protected SubscriptionApi subscriptionApi;

String externalKey = "somethingSpecial";
Subscription objFromJson = subscriptionApi.getSubscriptionByKey(externalKey, requestOptions);
```

```ruby
external_key = "somethingSpecial"
KillBillClient::Model::Subscription.find_by_external_key(external_key, options)
```

```python
subscriptionApi = killbill.api.SubscriptionApi()
external_key = 'somethingSpecial'
subscriptionApi.get_subscription_by_key(external_key, api_key, api_secret)
```

> Example Response:

```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 200 OK
< Content-Type: application/json
<
{
  "accountId": "1f979085-1765-471b-878a-5f640db4d831",
  "bundleId": "8815e8c0-afab-41b9-b793-cb8fef2382e4",
  "bundleExternalKey": "8815e8c0-afab-41b9-b793-cb8fef2382e4",
  "subscriptionId": "8e5c5339-1cad-46c6-ab18-3d5ddc1b2414",
  "externalKey": "somethingSpecial",
  "startDate": "2020-01-08",
  "productName": "Pistol",
  "productCategory": "BASE",
  "billingPeriod": "MONTHLY",
  "phaseType": "EVERGREEN",
  "priceList": "notrial",
  "planName": "pistol-monthly-notrial",
  "state": "ACTIVE",
  "sourceType": "NATIVE",
  "cancelledDate": null,
  "chargedThroughDate": "2020-05-08",
  "billingStartDate": "2020-01-08",
  "billingEndDate": null,
  "billCycleDayLocal": 8,
  "events": [
    {
      "eventId": "1d24928e-790d-4dc9-8a88-c4eaa56de392",
      "billingPeriod": "MONTHLY",
      "effectiveDate": "2020-01-08",
      "plan": "pistol-monthly-notrial",
      "product": "Pistol",
      "priceList": "notrial",
      "eventType": "START_ENTITLEMENT",
      "isBlockedBilling": false,
      "isBlockedEntitlement": false,
      "serviceName": "entitlement-service",
      "serviceStateName": "ENT_STARTED",
      "phase": "pistol-monthly-notrial-evergreen",
      "auditLogs": []
    },
    {
      "eventId": "3aaa3239-2bc6-4f04-977b-fce5de098af8",
      "billingPeriod": "MONTHLY",
      "effectiveDate": "2020-01-08",
      "plan": "pistol-monthly-notrial",
      "product": "Pistol",
      "priceList": "notrial",
      "eventType": "START_BILLING",
      "isBlockedBilling": false,
      "isBlockedEntitlement": false,
      "serviceName": "billing-service",
      "serviceStateName": "START_BILLING",
      "phase": "pistol-monthly-notrial-evergreen",
      "auditLogs": []
    }
  ],
  "priceOverrides": null,
  "prices": [
    {
      "planName": "pistol-monthly-notrial",
      "phaseName": "pistol-monthly-notrial-evergreen",
      "phaseType": "EVERGREEN",
      "fixedPrice": null,
      "recurringPrice": 19.95,
      "usagePrices": []
    }
  ],
  "auditLogs": []
}
```
```ruby
{
   "accountId":"0cdaeca7-4984-47dc-b245-7c32627f26cd",
   "bundleId":"d1f4ca8d-be47-4e64-84ce-f697b42d4182",
   "subscriptionId":"161692a4-c293-410c-a92f-939c5e3dcba7",
   "externalKey":"somethingSpecial",
   "bundleExternalKey":"a4f4ca8d-3447-4e64-84ce-6697b42d419c",
   "startDate":"2013-08-01",
   "productName":"Basic",
   "productCategory":"BASE",
   "billingPeriod":"MONTHLY",
   "phaseType":"EVERGREEN",
   "priceList":"DEFAULT",
   "planName":"basic-monthly-in-advance",
   "state":"ACTIVE",
   "sourceType":"NATIVE",
   "chargedThroughDate":"2013-09-01",
   "billingStartDate":"2013-08-01",
   "billCycleDayLocal":1,
   "events":[
      {
         "eventId":"dda11bf3-f74a-4c42-83e1-0f43a41389af",
         "billingPeriod":"MONTHLY",
         "effectiveDate":"2013-08-01",
         "plan":"basic-monthly-in-advance",
         "product":"Basic",
         "priceList":"DEFAULT",
         "eventType":"START_ENTITLEMENT",
         "isBlockedBilling":false,
         "isBlockedEntitlement":false,
         "serviceName":"entitlement-service",
         "serviceStateName":"ENT_STARTED",
         "phase":"basic-monthly-in-advance-evergreen",
         "auditLogs":[]
      },
      {
         "eventId":"6901117c-4ce0-4eb6-8642-380823490fae",
         "billingPeriod":"MONTHLY",
         "effectiveDate":"2013-08-01",
         "plan":"basic-monthly-in-advance",
         "product":"Basic",
         "priceList":"DEFAULT",
         "eventType":"START_BILLING",
         "isBlockedBilling":false,
         "isBlockedEntitlement":false,
         "serviceName":"billing-service",
         "serviceStateName":"START_BILLING",
         "phase":"basic-monthly-in-advance-evergreen",
         "auditLogs":[]
      }
   ],
   "prices":[
      {
         "planName":"basic-monthly-in-advance",
         "phaseName":"asic-monthly-in-advance-trial",
         "phaseType":"TRIAL",
         "fixedPrice":0,
         "recurringPrice":null,
         "usagePrices":[]
      },
      {
         "planName":"basic-monthly-in-advance",
         "phaseName":"basic-monthly-in-advance-evergreen",
         "phaseType":"EVERGREEN",
         "fixedPrice":null,
         "recurringPrice":500.0,
         "usagePrices":[]
      }
   ],
   "auditLogs":[]
}
```
```python
{'account_id': '3b1a5a67-f0ac-475c-9aad-735d309f0c1f',
 'audit_logs': [],
 'bill_cycle_day_local': 13,
 'billing_end_date': None,
 'billing_period': 'MONTHLY',
 'billing_start_date': datetime.date(2018, 5, 14),
 'bundle_id': 'e5590623-ccd4-4a8a-be26-008ce7c02b3b',
 'cancelled_date': None,
 'charged_through_date': None,
 'events': [{'audit_logs': [],
             'billing_period': 'MONTHLY',
             'effective_date': datetime.date(2018, 5, 14),
             'event_id': '46335d3d-8234-49c3-af1a-dcf8cd354ef3',
             'event_type': 'START_ENTITLEMENT',
             'is_blocked_billing': False,
             'is_blocked_entitlement': False,
             'phase': 'standard-monthly-trial',
             'plan': 'standard-monthly',
             'price_list': 'DEFAULT',
             'product': 'Standard',
             'service_name': 'entitlement-service',
             'service_state_name': 'ENT_STARTED'},
            {'audit_logs': [],
             'billing_period': 'MONTHLY',
             'effective_date': datetime.date(2018, 5, 14),
             'event_id': 'a14d6512-7479-46e0-b72b-81dff575d1d4',
             'event_type': 'START_BILLING',
             'is_blocked_billing': False,
             'is_blocked_entitlement': False,
             'phase': 'standard-monthly-trial',
             'plan': 'standard-monthly',
             'price_list': 'DEFAULT',
             'product': 'Standard',
             'service_name': 'billing-service',
             'service_state_name': 'START_BILLING'},
            {'audit_logs': [],
             'billing_period': 'MONTHLY',
             'effective_date': datetime.date(2018, 6, 13),
             'event_id': '27408c65-46b8-4bc9-a7ee-c80d6e5fb9b5',
             'event_type': 'PHASE',
             'is_blocked_billing': False,
             'is_blocked_entitlement': False,
             'phase': 'standard-monthly-evergreen',
             'plan': 'standard-monthly',
             'price_list': 'DEFAULT',
             'product': 'Standard',
             'service_name': 'entitlement+billing-service',
             'service_state_name': 'PHASE'}],
 'external_key': 'somethingSpecial',
 'phase_type': 'TRIAL',
 'plan_name': 'standard-monthly',
 'price_list': 'DEFAULT',
 'prices': [{'fixed_price': 0.0,
              'phase_name': 'standard-monthly-trial',
              'phase_type': 'TRIAL',
              'plan_name': 'standard-monthly',
              'recurring_price': None,
              'usage_prices': []},
             {'fixed_price': None,
              'phase_name': 'standard-monthly-evergreen',
              'phase_type': 'EVERGREEN',
              'plan_name': 'standard-monthly',
              'recurring_price': 100.0,
              'usage_prices': []}],
 'product_category': 'BASE',
 'product_name': 'Standard',
 'source_type': 'NATIVE',
 'start_date': datetime.date(2018, 5, 14),
 'state': 'ACTIVE',
 'subscription_id': '4aab9b96-c2e7-4641-a6d9-db984969201e'}
```

**Query Parameters**

| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| **externalKey** | String | true | the subscription external key |
| **audit** | enum | false | level of audit logs returned |

**Returns**

Returns a subscription object.


### Update the BCD associated to a subscription

This allows to change the Bill Cycle Date, BCD, for a given subscription.
This only apply for subscriptions whose recurring term is month based -- e.g `MONTHLY`, `ANNUAL`, ... 

For example if a given subscription was invoiced on the 1st, then one could use this api
to realign invoicing, let's say on the 16th. 

**HTTP Request** 

`PUT http://127.0.0.1:8080/1.0/kb/subscriptions/{subscriptionId}/bcd`

> Example Request:

```shell
curl -v \
    -X PUT \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Content-Type: application/json" \
    -H "X-Killbill-CreatedBy: demo" \
    -d '{ "billCycleDayLocal": 16 }' \
    'http://127.0.0.1:8080/1.0/kb/subscriptions/77e23878-8b9d-403b-bf31-93003e125712/bcd' 
```

```java
import org.killbill.billing.client.api.gen.SubscriptionApi;
protected SubscriptionApi subscriptionApi;

UUID subscriptionId = UUID.fromString("905a0636-ab63-40c0-acd4-b461b6808b5d");

Subscription updatedSubscription = new Subscription();
updatedSubscription.setSubscriptionId(subscriptionId);
updatedSubscription.setBillCycleDayLocal(9);

LocalDate effectiveFromDate = null;

subscriptionApi.updateSubscriptionBCD(subscriptionId, 
                                      updatedSubscription, 
                                      effectiveFromDate, 
                                      requestOptions);
```

```ruby
subscription                      = KillBillClient::Model::Subscription.new
subscription.subscription_id      = "161692a4-c293-410c-a92f-939c5e3dcba7"
subscription.bill_cycle_day_local = 16

effective_from_date  = '2018-08-16'
force_past_effective_date = nil

subscription.update_bcd(user, 
                        reason, 
                        comment, 
                        effective_from_date, 
                        force_past_effective_date, 
                        options)
```

```python
subscriptionApi = killbill.api.SubscriptionApi()
subscription_id = '161692a4-c293-410c-a92f-939c5e3dcba7'
body = Subscription(subscription_id=subscription_id,
                    bill_cycle_day_local=26)

subscriptionApi.update_subscription_bcd(subscription_id,
                                        body,
                                        created_by,
                                        api_key,
                                        api_secret)
```

> Example Response:

```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 204 No Content
< Content-Type: application/json
< Content-Length: 0
```

```java
no content
```
```ruby
no content
```
```python
no content
```

**Query Parameters**

| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| **effectiveFromDate** | string | true | effective at which this change becomes effective, a null value means immediate | 
| **forceNewBcdWithPastEffectiveDate** | boolean | false | by default the effective date must be in the future so as to not modify existing invoices. this flag allows to override this behavior. (default: false)| 

**Returns**

A `204` http status without content.


### Change subscription plan


Ability to upgrade/downgrade a given subscription to a new `Plan`.

**HTTP Request** 

`PUT http://127.0.0.1:8080/1.0/kb/subscriptions/{subscriptionId}`

> Example Request:

```shell
curl -v \
    -X PUT \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Content-Type: application/json" \
    -H "X-Killbill-CreatedBy: demo" \
    -d '{ 
            "productName": "Sports", 
            "billingPeriod": "MONTHLY", 
            "priceList": "DEFAULT"
        }' \
    'http://127.0.0.1:8080/1.0/kb/subscriptions/77e23878-8b9d-403b-bf31-93003e125712'
```

```java
import org.killbill.billing.client.api.gen.SubscriptionApi;
protected SubscriptionApi subscriptionApi;

UUID subscriptionId = UUID.fromString("905a0636-ab63-40c0-acd4-b461b6808b5d");

Subscription newInput = new Subscription();
newInput.setSubscriptionId(subscriptionId);

// Specify the product, billing period and price list
newInput.setProductName("Shotgun");
newInput.setBillingPeriod(BillingPeriod.MONTHLY);
newInput.setPriceList("DEFAULT");

// Alternatively, you can specify the plan name
newInput.setPlanName("shotgun-monthly");

LocalDate requestedDate = null;

subscriptionApi.changeSubscriptionPlan(subscriptionId, 
                                       newInput, 
                                       requestedDate, 
                                       BillingActionPolicy.IMMEDIATE, 
                                       NULL_PLUGIN_PROPERTIES, 
                                       requestOptions);
```

```ruby
input = {
           :productName => 'Super', 
           :billingPeriod => 'MONTHLY', 
           :priceList => 'DEFAULT'
         }
         
requested_date = nil
billing_policy = nil
call_completion = false
target_phase_type = nil

subscription.change_plan(input, 
                         user, 
                         reason, 
                         comment, 
                         requested_date, 
                         billing_policy,
                         target_phase_type, 
                         call_completion, 
                         options)
```

```python
subscriptionApi = killbill.api.SubscriptionApi()
subscription_id = '97278000-72fd-45d7-9b67-e44690bdb074'
body = Subscription(product_name='Super',
                    billing_period='MONTHLY',
                    price_list='DEFAULT')

subscriptionApi.change_subscription_plan(subscription_id,
                                         body,
                                         created_by,
                                         api_key,
                                         api_secret)
```

> Example Response:

```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 204 No Content
< Content-Type: application/json
< Content-Length: 0
```

```java
no content
```
```ruby
{
   "accountId":"986c5d4e-b322-4d71-ad24-e3bf6e38734a",
   "bundleId":"b0b9da5f-6844-417b-ac97-d7e8df07c26a",
   "subscriptionId":"97278000-72fd-45d7-9b67-e44690bdb074",
   "externalKey":"986c5d4e-b322-4d71-ad24-e3bf6e38734a-452347",
   "bundleExternalKey":"765c5d4e-b322-4d71-6544-e3bf6e38734a-cd2347",
   "startDate":"2013-08-01",
   "productName":"Super",
   "productCategory":"BASE",
   "billingPeriod":"MONTHLY",
   "phaseType":"TRIAL",
   "priceList":"DEFAULT",
   "planName":"super-monthly",
   "state":"ACTIVE",
   "sourceType":"NATIVE",
   "chargedThroughDate":"2013-08-01",
   "billingStartDate":"2013-08-01",
   "billCycleDayLocal":31,
   "events":[
      {
         "eventId":"b2c4195a-0888-44e4-91a7-537b20b08bd8",
         "billingPeriod":"MONTHLY",
         "effectiveDate":"2013-08-01",
         "plan":"sports-monthly",
         "product":"Sports",
         "priceList":"DEFAULT",
         "eventType":"START_ENTITLEMENT",
         "isBlockedBilling":false,
         "isBlockedEntitlement":false,
         "serviceName":"entitlement-service",
         "serviceStateName":"ENT_STARTED",
         "phase":"sports-monthly-trial",
         "auditLogs":[]
      },
      {
         "eventId":"0c15d9ba-f0f3-40e6-83d7-2d84af703d06",
         "billingPeriod":"MONTHLY",
         "effectiveDate":"2013-08-01",
         "plan":"sports-monthly",
         "product":"Sports",
         "priceList":"DEFAULT",
         "eventType":"START_BILLING",
         "isBlockedBilling":false,
         "isBlockedEntitlement":false,
         "serviceName":"billing-service",
         "serviceStateName":"START_BILLING",
         "phase":"sports-monthly-trial",
         "auditLogs":[]
      },
      {
         "eventId":"b675c39f-eeef-4cc1-8ffc-e50f51e8a84a",
         "billingPeriod":"MONTHLY",
         "effectiveDate":"2013-08-02",
         "plan":"super-monthly",
         "product":"Super",
         "priceList":"DEFAULT",
         "eventType":"CHANGE",
         "isBlockedBilling":false,
         "isBlockedEntitlement":false,
         "serviceName":"entitlement+billing-service",
         "serviceStateName":"CHANGE",
         "phase":"super-monthly-trial",
         "auditLogs":[]
      },
      {
         "eventId":"e411fe3e-6d38-4256-bd96-867f3a50c634",
         "billingPeriod":"MONTHLY",
         "effectiveDate":"2013-08-31",
         "plan":"super-monthly",
         "product":"Super",
         "priceList":"DEFAULT",
         "eventType":"PHASE",
         "isBlockedBilling":false,
         "isBlockedEntitlement":false,
         "serviceName":"entitlement+billing-service",
         "serviceStateName":"PHASE",
         "phase":"super-monthly-evergreen",
         "auditLogs":[]
      }
   ],
   "prices":[
      {
         "planName":"sports-monthly",
         "phaseName":"sports-monthly-trial",
         "phaseType":"TRIAL",
         "fixedPrice":0,
         "recurringPrice":null,
         "usagePrices":[]
      },
      {
         "planName":"sports-monthly",
         "phaseName":"sports-monthly-evergreen",
         "phaseType":"EVERGREEN",
         "fixedPrice":null,
         "recurringPrice":500.0,
         "usagePrices":[]
      }
   ],
   "auditLogs":[]
}
```
```python
no content
```

**Query Parameters**

| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| **billingPolicy** | string | false | billing policy that will be used to make this change effective -- e.g `END_OF_TERM` would ensure there is no pro-ratin. |
| **requestedDate** | string | false | the date at which this change should become effective. This date is only used if no billingPolicy was specified. A null date makes it an immediate change. |
| **callCompletion** | boolean | false | ability to block the call until invoice and payment, if any, have been generated. (default: false)|
| **callTimeoutSec** | long | false | when setting callCompletion=true, timeout in sec |

**Returns**

A `204` http status without content.


### Undo a pending change plan on a subscription

This endpoint allows to undo a pending change of `Plan` for a given subscription.

**HTTP Request** 

`PUT http://127.0.0.1:8080/1.0/kb/subscriptions/{subscriptionId}/undoChangePlan`

> Example Request:

```shell
curl \
    -X PUT \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "X-Killbill-CreatedBy: demo" \
    'http://127.0.0.1:8080/1.0/kb/subscriptions/77e23878-8b9d-403b-bf31-93003e125712/undoChangePlan' 
```

```java
import org.killbill.billing.client.api.gen.SubscriptionApi;
protected SubscriptionApi subscriptionApi;

UUID subscriptionId = UUID.fromString("1bb4b638-3886-4f73-90a5-89eb6d1bcf7f");
ImmutableMap<String, String> NULL_PLUGIN_PROPERTIES = null;

subscriptionApi.undoChangeSubscriptionPlan(subscriptionId, 
                                           NULL_PLUGIN_PROPERTIES, 
                                           requestOptions);
```

```ruby
subscription.undo_change_plan(user, 
                              reason, 
                              comment, 
                              options)
```

```python
subscriptionApi = killbill.api.SubscriptionApi()
subscription_id = 'f5bb14ed-c6e8-4895-8d4e-34422e12cdfa'

subscriptionApi.undo_change_subscription_plan(subscription_id,
                                              created_by,
                                              api_key,
                                              api_secret)
```

> Example Response:

```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 204 No Content
< Content-Type: application/json
< Content-Length: 0
```

```java
no content
```
```ruby
no content
```
```python
no content
```

**Query Parameters**

None. 

**Returns**

A `204` http status without content.



### Cancel a subscription plan

**HTTP Request** 

`DELETE http://127.0.0.1:8080/1.0/kb/subscriptions/{subscriptionId}`

> Example Request:

```shell
curl  -v \
    -X DELETE \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "X-Killbill-CreatedBy: demo" \
    'http://127.0.0.1:8080/1.0/kb/subscriptions/77e23878-8b9d-403b-bf31-93003e125712'
```

```java
import org.killbill.billing.client.api.gen.SubscriptionApi;
protected SubscriptionApi subscriptionApi;

UUID accountId = UUID.fromString("905a0636-ab63-40c0-acd4-b461b6808b5d");
LocalDate requestedDate = null;
EntitlementActionPolicy entitlementPolicy = null;
BillingActionPolicy billingPolicy = null;
ImmutableMap<String, String> NULL_PLUGIN_PROPERTIES = null;

subscriptionApi.cancelSubscriptionPlan(accountId, 
                                       requestedDate, 
                                       entitlementPolicy, 
                                       billingPolicy, 
                                       NULL_PLUGIN_PROPERTIES, 
                                       requestOptions);
```

```ruby
requested_date = nil
entitlement_policy = nil
billing_policy = nil
use_requested_date_for_billing = nil

subscription.cancel(user, 
                    reason, 
                    comment, 
                    requested_date, 
                    entitlement_policy, 
                    billing_policy, 
                    use_requested_date_for_billing, 
                    options)
```

```python
subscriptionApi = killbill.api.SubscriptionApi()
subscription_id = 'ee508b5b-46b8-42a7-8988-16c0470de4ae'

subscriptionApi.cancel_subscription_plan(subscription_id, 
                                         created_by,
                                         api_key,
                                         api_secret)
```

> Example Response:

```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 204 No Content
< Content-Type: application/json
< Content-Length: 0
```

```java
no content
```
```ruby
no content
```
```python
no content
```

**Query Parameters**

| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| **requestedDate** | string | false | requested date. A null value means immediately. |
| **entitlementPolicy** | string | false | entitlement policy |
| **billingPolicy** | string | false | billing policy |
| **useRequestedDateForBilling** | boolean | false | use requested date for billing (default: false) |
| **callCompletion** | boolean | false | ability to block the call until invoice and payment, if any, have been generated. (default: false)|
| **callTimeoutSec** | long | false | when setting callCompletion=true, timeout in sec |

Since we offer the ability to control cancelation date for both entitlement (service) and billing either through policies, dates or null values (now), it is imperative to understand how those parameters work:

* If `entitlementPolicy` has been defined, the `requestedDate` is ignored, and we either default to the catalog defined `billingPolicy` for this `Plan`, or use the one provided in this api.
* If not, the `requestedDate` is used to compute the entitlement cancelation date, and the null value value means change should be immediate. The billing date will then be computed the following way:
  * If `billingPolicy` has been specified, it is used to compute the billing cancelation date
  * If `billingPolicy` has not been specified, we either use the `requestedDate` when `useRequestedDateForBilling` is true or default to the catalog defined `billingPolicy` for this `Plan`

So, the common use case would require the following:

* **Immediate cancelation**: This will create a pro-ration credit unless this aligns with the subscription charged through date (`CTD`), that is, the date up to which it was invoiced, and then from an entitlement point of view it will deactivate service immediately. In order to achieve this result, one can pass the following parameters: `entitlementPolicy`=`IMMEDIATE` and `billingPolicy`=`IMMEDIATE`; alternatively passing no parameters and therefore a null `requestedDate` would produce the same result.
* **EOT cancelation**: This will not create any pro-ration and keep the service active until the end of the period (`CTD`). In order to achieve this result, one can pass the following parameters: `entitlementPolicy`=`END_OF_TERM` and `billingPolicy`=`END_OF_TERM`.

The reason for all this complexity is to allow to control entitlement and billing date separately, and also avoid users to have to compute dates to achieve certain behavior by relying on well defined policies.


**Returns**

A `204` http status without content.


### Un-cancel a subscription

This endpoint allows to undo a pending cancelation for a given subscription.

**HTTP Request** 

`PUT http://127.0.0.1:8080/1.0/kb/subscriptions/{subscriptionId}/uncancel`

> Example Request:

```shell
curl \
    -X PUT \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "X-Killbill-CreatedBy: demo" \
    'http://127.0.0.1:8080/1.0/kb/subscriptions/77e23878-8b9d-403b-bf31-93003e125712/uncancel' 
```

```java
import org.killbill.billing.client.api.gen.SubscriptionApi;
protected SubscriptionApi subscriptionApi;

UUID subscriptionId = UUID.fromString("1bb4b638-3886-4f73-90a5-89eb6d1bcf7f");
ImmutableMap<String, String> NULL_PLUGIN_PROPERTIES = null;

subscriptionApi.uncancelSubscriptionPlan(subscriptionId, 
                                         NULL_PLUGIN_PROPERTIES, 
                                         requestOptions);
```

```ruby
subscription.uncancel(user, 
                      reason, 
                      comment, 
                      options)
```

```python
subscriptionApi = killbill.api.SubscriptionApi()
subscription_id = 'f5bb14ed-c6e8-4895-8d4e-34422e12cdfa'

subscriptionApi.uncancel_subscription_plan(subscription_id,
                                           created_by,
                                           api_key,
                                           api_secret)
```

> Example Response:

```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 204 No Content
< Content-Type: application/json
< Content-Length: 0
```

```java
no content
```
```ruby
no content
```
```python
no content
```

**Query Parameters**

None. 

**Returns**

A `204` http status without content.


## Blocking State

See section [Account Blocing State](#account-blocking-state) for an introduction.

### Block a subscription

Provides an low level interface to add `BlockingState` event for this subscription. 

**HTTP Request** 

`POST http://127.0.0.1:8080/1.0/kb/subscriptions/{subscriptionId}/block`

> Example Request:

```shell
curl -v \
    -X POST \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Content-Type: application/json" \
    -H "X-Killbill-CreatedBy: demo" \
    -d '{ 
            "stateName": "STATE",
            "service": "ServiceStateService",
            "isBlockChange": false,
            "isBlockEntitlement": false,
            "isBlockBilling": false
    }' \
    'http://127.0.0.1:8080/1.0/kb/subscriptions/77e23878-8b9d-403b-bf31-93003e125712/block' 
```

```java
import org.killbill.billing.client.api.gen.SubscriptionApi;
protected SubscriptionApi subscriptionApi;

UUID subscriptionId = UUID.fromString("161692a4-c293-410c-a92f-939c5e3dcba7");

BlockingState blockingState = new BlockingState();
blockingState.setStateName("STATE1");
blockingState.setService("ServiceStateService");
blockingState.setIsBlockChange(false);
blockingState.setIsBlockBilling(false);
blockingState.setIsBlockEntitlement(false);

LocalDate requestedDate = new LocalDate("2013-08-01");
Map<String, String> pluginProperty = ImmutableMap.<String, String>of();

BlockingStates result = subscriptionApi.addSubscriptionBlockingState(subscriptionId,
                                                                     blockingState,
                                                                     requestedDate,
                                                                     pluginProperty,
                                                                     requestOptions);
```

```ruby
subscription = KillBillClient::Model::Subscription.new
subscription.subscription_id = "161692a4-c293-410c-a92f-939c5e3dcba7"

state_name = 'STATE1'
service = 'ServiceStateService'
block_change = false
block_entitlement = false
block_billing = false
requested_date = nil

subscription.set_blocking_state(state_name, 
                                service, 
                                block_change, 
                                block_entitlement, 
                                block_billing, 
                                requested_date, 
                                user, 
                                reason,
                                comment, 
                                options)
```

```python
subscriptionApi = killbill.api.SubscriptionApi()
body = BlockingState(state_name='STATE1',
                     service='ServiceStateService',
                     is_block_change=False,
                     is_block_entitlement=False,
                     is_block_billing=False)

subscriptionApi.add_subscription_blocking_state(subscription_id,
                                                body,
                                                created_by,
                                                api_key,
                                                api_secret)
```

> Example Response:

```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 204 No Content
< Content-Type: application/json
< Location: http://127.0.0.1:8080/1.0/kb/accounts/581d86fc-7cfc-46f2-b6d4-4dbc1d98beb3/block?blockingStateTypes=SUBSCRIPTION
< Content-Length: 0
```
```java
[class BlockingState {
    org.killbill.billing.client.model.gen.BlockingState@bf898dea
    blockedId: 161692a4-c293-410c-a92f-939c5e3dcba7
    stateName: ENT_STARTED
    service: entitlement-service
    isBlockChange: false
    isBlockEntitlement: false
    isBlockBilling: false
    effectiveDate: 2012-04-25T00:03:42.000Z
    type: SUBSCRIPTION
    auditLogs: []
}, class BlockingState {
    org.killbill.billing.client.model.gen.BlockingState@fd3732d8
    blockedId: 161692a4-c293-410c-a92f-939c5e3dcba7
    stateName: STATE1
    service: ServiceStateService
    isBlockChange: false
    isBlockEntitlement: false
    isBlockBilling: false
    effectiveDate: 2013-08-01T00:03:42.000Z
    type: SUBSCRIPTION
    auditLogs: []
}]
```
```ruby
no content
```
```python
no content
```

**Query Parameters**

| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| **requestedDate** | string | false | requested date | 

**Returns**

A `201` http status without content.


## Custom Fields

Custom fields are `{key, value}` attributes that can be attached to any customer resources, and in particularly on the `Subscription` objects.

### Add custom fields to subscription

**HTTP Request** 

`POST http://127.0.0.1:8080/1.0/kb/subscriptions/{subscriptionId}/customFields`

> Example Request:

```shell
curl -v \
    -X POST \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Content-Type: application/json" \
    -H "X-Killbill-CreatedBy: demo" \
    -d '[{ 
            "objectId": "77e23878-8b9d-403b-bf31-93003e125712",
            "objectType": "SUBSCRIPTION",
            "name": "Test Custom Field",
            "value": "test_value"
    }]' \
    'http://127.0.0.1:8080/1.0/kb/subscriptions/77e23878-8b9d-403b-bf31-93003e125712/customFields' 
```

```java
import org.killbill.billing.client.api.gen.SubscriptionApi;
protected SubscriptionApi subscriptionApi;

UUID subscriptionId = UUID.fromString("cca08349-8b26-41c7-bfcc-2e3cf70a0f28");

final ImmutableList<AuditLog> EMPTY_AUDIT_LOGS = ImmutableList.<AuditLog>of();

CustomFields customFields = new CustomFields();
customFields.add(new CustomField(null, 
                                 subscriptionId, 
                                 ObjectType.SUBSCRIPTION, 
                                 "Test Custom Field", 
                                 "test_value", 
                                 EMPTY_AUDIT_LOGS));

subscriptionApi.createSubscriptionCustomFields(subscriptionId, 
                                               customFields, 
                                               requestOptions);
```

```ruby
custom_field = KillBillClient::Model::CustomFieldAttributes.new
custom_field.object_type = 'SUBSCRIPTION'
custom_field.name = 'Test Custom Field'
custom_field.value = 'test_value'

subscription.add_custom_field(custom_field, 
                              user,
                              reason,
                              comment,
                              options)
```

```python
subscriptionApi = killbill.api.SubscriptionApi()
subscription_id = '4927c1a2-3959-4f71-98e7-ce3ba19c92ac'
body = CustomField(name='Test Custom Field', value='test_value')

subscriptionApi.create_subscription_custom_fields(subscription_id,
                                                  [body],
                                                  created_by,
                                                  api_key,
                                                  api_secret)
```

> Example Response:

```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 201 Created
< Location: http://127.0.0.1:8080/1.0/kb/subscriptions/77e23878-8b9d-403b-bf31-93003e125712/customFields
< Content-Type: application/json
< Content-Length: 0
```

```java
//First element of the list
class CustomField {
    org.killbill.billing.client.model.gen.CustomField@c7d0c38a
    customFieldId: null
    objectId: cca08349-8b26-41c7-bfcc-2e3cf70a0f28
    objectType: SUBSCRIPTION
    name: Test Custom Field
    value: test_value
    auditLogs: []
}
```
```ruby
[
   {
      "customFieldId":"7fb3dde7-0911-4477-99e3-69d142509bb9",
      "objectId":"4927c1a2-3959-4f71-98e7-ce3ba19c92ac",
      "objectType":"SUBSCRIPTION",
      "name":"Test Custom Field",
      "value":"test_value",
      "auditLogs":[]
   }
]
```

```python
no content
```


**Query Parameters**

None.

**Returns**

Returns a custom field object.

### Retrieve subscription custom fields

**HTTP Request** 

`GET http://127.0.0.1:8080/1.0/kb/subscriptions/{subscriptionId}/customFields`

> Example Request:

```shell
curl \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Accept: application/json" \
    'http://127.0.0.1:8080/1.0/kb/subscriptions/77e23878-8b9d-403b-bf31-93003e125712/customFields' 
```

```java
import org.killbill.billing.client.api.gen.SubscriptionApi;
protected SubscriptionApi subscriptionApi;

UUID subscriptionId = UUID.fromString("cca08349-8b26-41c7-bfcc-2e3cf70a0f28");

List<CustomField> customFields = subscriptionApi.getSubscriptionCustomFields(subscriptionId,
                                                                             AuditLevel.NONE,
                                                                             requestOptions);
```

```ruby
audit = 'NONE'

subscription.custom_fields(audit, options)
```

```python
subscriptionApi = killbill.api.SubscriptionApi()
subscription_id = '642ee0ac-972b-4cdf-b9ae-ab8f9bb9bc05'

subscriptionApi.get_subscription_custom_fields(subscription_id,
                                               api_key,
                                               api_secret)
```

> Example Response:

```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 200 Ok
< Content-Type: application/json
<
[
    {
        "customFieldId":"439ed0f8-9b37-4688-bace-e2595b1d3801",
        "objectId":"77e23878-8b9d-403b-bf31-93003e125712",
        "objectType":"SUBSCRIPTION",
        "name":"Test Custom Field",
        "value":"test_value",
        "auditLogs":[]
    }
]
```

```java
//First element of the list
class CustomField {
    org.killbill.billing.client.model.gen.CustomField@c7d0c38a
    customFieldId: null
    objectId: cca08349-8b26-41c7-bfcc-2e3cf70a0f28
    objectType: SUBSCRIPTION
    name: Test Custom Field
    value: test_value
    auditLogs: []
}
```
```ruby
[
   {
      "customFieldId":"7fb3dde7-0911-4477-99e3-69d142509bb9",
      "objectId":"4927c1a2-3959-4f71-98e7-ce3ba19c92ac",
      "objectType":"SUBSCRIPTION",
      "name":"Test Custom Field",
      "value":"test_value",
      "auditLogs":[]
   }
]
```

```python
[{'audit_logs': [],
 'custom_field_id': '31a21da4-1eae-4f83-b9e4-49c53217d33e',
 'name': 'Test Custom Field',
 'object_id': '642ee0ac-972b-4cdf-b9ae-ab8f9bb9bc05',
 'object_type': 'SUBSCRIPTION',
 'value': 'test_value'}]
```


**Query Parameters**

| Name | Type | Required | Description |
| ---- | -----| -------- | ----------- | 
| **audit** | enum | false | level of audit logs returned |

**Returns**

Returns a list of custom field objects.

### Modify custom fields to subscription

**HTTP Request** 

`PUT http://127.0.0.1:8080/1.0/kb/subscriptions/{subscriptionId}/customFields`

> Example Request:

```shell
curl -v \
    -X PUT \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Content-Type: application/json" \
    -H "X-Killbill-CreatedBy: demo" \
    -d '[{ 
            "customFieldId": "439ed0f8-9b37-4688-bace-e2595b1d3801",
            "value": "NewValue"
    }]' \
    'http://127.0.0.1:8080/1.0/kb/subscriptions/77e23878-8b9d-403b-bf31-93003e125712/customFields' 
```

```java
import org.killbill.billing.client.api.gen.SubscriptionApi;
protected SubscriptionApi subscriptionApi;

UUID subscriptionId = UUID.fromString("cca08349-8b26-41c7-bfcc-2e3cf70a0f28");
UUID customFieldsId = UUID.fromString("9913e0f6-b5ef-498b-ac47-60e1626eba8f");

CustomField customFieldModified = new CustomField();
customFieldModified.setCustomFieldId(customFieldsId);
customFieldModified.setValue("NewValue");

subscriptionApi.modifySubscriptionCustomFields(subscriptionId, 
                                               customFieldModified, 
                                               requestOptions);
```

```ruby
custom_field.custom_field_id = '7fb3dde7-0911-4477-99e3-69d142509bb9'
custom_field.name = 'Test Modify'
custom_field.value = 'test_modify_value'

subscription.modify_custom_field(custom_field,                                                                                            
                                 user, 
                                 reason,
                                 comment, 
                                 options)
```

```python
subscriptionApi = killbill.api.SubscriptionApi()
subscription_id = '4927c1a2-3959-4f71-98e7-ce3ba19c92ac'
custom_field_id = '7fb3dde7-0911-4477-99e3-69d142509bb9'
body = CustomField(custom_field_id=custom_field_id, 
                   name='Test Custom Field', 
                   value='test_value')

subscriptionApi.modify_subscription_custom_fields(subscription_id,
                                                  [body],
                                                  created_by,
                                                  api_key,
                                                  api_secret)
```

> Example Response:

```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 204 No Content
< Content-Type: application/json
< Content-Length: 0
```

```java
no content
```

```ruby
no content
```

```python
no content
```


**Query Parameters**

None.

**Returns**

A `204` http status without content.

### Remove custom fields from subscription

**HTTP Request** 

`DELETE http://127.0.0.1:8080/1.0/kb/subscriptions/{subscriptionId}/customFields`

> Example Request:

```shell
curl -v \
    -X DELETE \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "X-Killbill-CreatedBy: demo" \
    'http://127.0.0.1:8080/1.0/kb/subscriptions/77e23878-8b9d-403b-bf31-93003e125712/customFields?customField=439ed0f8-9b37-4688-bace-e2595b1d3801' 
```

```java
import org.killbill.billing.client.api.gen.SubscriptionApi;
protected SubscriptionApi subscriptionApi;

UUID subscriptionId = UUID.fromString("cca08349-8b26-41c7-bfcc-2e3cf70a0f28");
UUID customFieldsId = UUID.fromString("9913e0f6-b5ef-498b-ac47-60e1626eba8f");

subscriptionApi.deleteSubscriptionCustomFields(subscriptionId, 
                                               customFieldsId, 
                                               requestOptions);
```

```ruby
custom_field_id = custom_field.id

subscription.remove_custom_field(custom_field_id,                                                                                            
                                 user, 
                                 reason,
                                 comment, 
                                 options)
```

```python
subscriptionApi = killbill.api.SubscriptionApi()
subscription_id = '4927c1a2-3959-4f71-98e7-ce3ba19c92ac'

subscriptionApi.delete_subscription_custom_fields(subscription_id,
                                                  created_by,
                                                  api_key,
                                                  api_secret)
```

> Example Response:

```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 204 No Content
< Content-Type: application/json
< Content-Length: 0
```

```java
no content
```
```ruby
no content
```
```python
no content
```


**Query Parameters**

| Name | Type | Required | Description |
| ---- | -----| -------- | ----------- | 
| **customField** | string | true | the list of custom field object IDs that should be deleted. |

**Returns**

A `204` http status without content.

## Tags


See section [Account Tags](#account-tags) for an introduction.

The are no `system` tags applicable for an `Subscription`.

Let's assume there is an existing `user` tagDefintion already created with `tagDefinitionId`=`353752dd-9041-4450-b782-a8bb03a923c8`.

### Add tags to subscription

**HTTP Request** 

`POST http://127.0.0.1:8080/1.0/kb/subscriptions/{subscriptionId}/tags`

> Example Request:

```shell
curl -v \
    -X POST \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Content-Type: application/json" \
    -H "X-Killbill-CreatedBy: demo" \
    -d '[
            "353752dd-9041-4450-b782-a8bb03a923c8"
        ]' \
  'http://127.0.0.1:8080/1.0/kb/subscriptions/77e23878-8b9d-403b-bf31-93003e125712/tags' 
```

```java
import org.killbill.billing.client.api.gen.SubscriptionApi;
protected SubscriptionApi subscriptionApi;

UUID subscriptionId = UUID.fromString("1bb4b638-3886-4f73-90a5-89eb6d1bcf7f");
UUID tagDefinitionId = UUID.fromString("353752dd-9041-4450-b782-a8bb03a923c8");

Tags result = subscriptionApi.createSubscriptionTags(subscriptionId, 
                                                     ImmutableList.<UUID>of(tagDefinitionId), 
                                                     requestOptions);
```

```ruby
tag_name = 'foo'

subscription.add_tag(tag_name,
                     user,
                     reason,
                     comment,
                     options)
```

```python
subscriptionApi = killbill.api.SubscriptionApi()
subscription_id = '28af3cb9-275b-4ac4-a55d-a0536e479069'
tag = ["353752dd-9041-4450-b782-a8bb03a923c8"]

subscriptionApi.create_subscription_tags(subscription_id,
                                         tag,
                                         created_by,
                                         api_key,
                                         api_secret)
```
> Example Response:

```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 201 Created
< Location: http://127.0.0.1:8080/1.0/kb/subscriptions/77e23878-8b9d-403b-bf31-93003e125712/tags
< Content-Type: application/json
< Content-Length: 0
```

```java
//First element of the list
class Tag {
    org.killbill.billing.client.model.gen.Tag@bd138472
    tagId: 1bb4b638-3886-4f73-90a5-89eb6d1bcf7f
    objectType: SUBSCRIPTION
    objectId: 917992d3-5f1f-4828-9fff-799cc4211aa9
    tagDefinitionId: 353752dd-9041-4450-b782-a8bb03a923c8
    tagDefinitionName: foo
    auditLogs: []
}
```
```ruby
[
   {
      "tagId":"a46cfeb6-e175-42db-be62-7f117326ab4e",
      "objectType":"SUBSCRIPTION",
      "objectId":"28af3cb9-275b-4ac4-a55d-a0536e479069",
      "tagDefinitionId":"353752dd-9041-4450-b782-a8bb03a923c8",
      "tagDefinitionName":"foo",
      "auditLogs":[]
   }
]
```
```python
no content
```

**Query Parameters**

None.

**Returns**

A `201` http status without content.

### Retrieve subscription tags

**HTTP Request** 

`GET http://127.0.0.1:8080/1.0/kb/subscriptions/{subscriptionId}/tags`

> Example Request:

```shell
curl \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Accept: application/json" \
    'http://127.0.0.1:8080/1.0/kb/subscriptions/77e23878-8b9d-403b-bf31-93003e125712/tags' 
```

```java
import org.killbill.billing.client.api.gen.SubscriptionApi;
protected SubscriptionApi subscriptionApi;

UUID subscriptionId = UUID.fromString("1bb4b638-3886-4f73-90a5-89eb6d1bcf7f");

Boolean includedDeleted = false; // Will not include deleted tags

List<Tag> tags = subscriptionApi.getSubscriptionTags(subscriptionId, 
                                                     includedDeleted, 
                                                     AuditLevel.FULL, 
                                                     requestOptions);
```

```ruby
included_deleted = false
audit = 'NONE'

subscription.tags(included_deleted,
                  audit,
                  options)
```

```python
subscriptionApi = killbill.api.SubscriptionApi()
subscription_id = 'f5bb14ed-c6e8-4895-8d4e-34422e12cdfa'

subscriptionApi.get_subscription_tags(subscription_id,
                                      api_key,
                                      api_secret)
```

> Example Response:

```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 200 OK
<
[
    {
        "tagId":"d0513754-56a9-4694-abb1-3ac46c72e861",
        "objectType":"SUBSCRIPTION",
        "objectId":"77e23878-8b9d-403b-bf31-93003e125712",
        "tagDefinitionId":"353752dd-9041-4450-b782-a8bb03a923c8",
        "tagDefinitionName":"foo",
        "auditLogs":[]
    }
]
```

```java
//First element of the list
class Tag {
    org.killbill.billing.client.model.gen.Tag@bd138472
    tagId: 1bb4b638-3886-4f73-90a5-89eb6d1bcf7f
    objectType: SUBSCRIPTION
    objectId: 917992d3-5f1f-4828-9fff-799cc4211aa9
    tagDefinitionId: 353752dd-9041-4450-b782-a8bb03a923c8
    tagDefinitionName: foo
    auditLogs: []
}
```
```ruby
[
   {
      "tagId":"a46cfeb6-e175-42db-be62-7f117326ab4e",
      "objectType":"SUBSCRIPTION",
      "objectId":"28af3cb9-275b-4ac4-a55d-a0536e479069",
      "tagDefinitionId":"353752dd-9041-4450-b782-a8bb03a923c8",
      "tagDefinitionName":"foo",
      "auditLogs":[]
   }
]
```
```python
[{'audit_logs': [],
 'object_id': 'f5bb14ed-c6e8-4895-8d4e-34422e12cdfa',
 'object_type': 'SUBSCRIPTION',
 'tag_definition_id': '353752dd-9041-4450-b782-a8bb03a923c8',
 'tag_definition_name': 'foo',
 'tag_id': 'a1fd0122-1ec8-4bc3-b71e-ab2a76ae5957'}]
```

**Query Parameters**

| Name | Type | Required | Description |
| ---- | -----| -------- | ---- | ------------
| **audit** | enum | false | level of audit logs returned |
| **includedDeleted** | boolean | false | choose true if you want to include deleted tags |

**Returns**

Returns a list of bundle tag objects.

### Remove tags from subscription

**HTTP Request** 

`DELETE http://127.0.0.1:8080/1.0/kb/subscriptions/{subscriptionId}/tags`

**Query Parameters**

| Name | Type | Required | Description |
| ---- | -----| -------- | ---- | ------------
| **tagDef** | string | true | the list of tag definition ID identifying the tags that should be removed. |

> Example Request:

```shell
curl -v \
    -X DELETE \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "X-Killbill-CreatedBy: demo" \
    'http://127.0.0.1:8080/1.0/kb/subscriptions/77e23878-8b9d-403b-bf31-93003e125712/tags?tagDef=353752dd-9041-4450-b782-a8bb03a923c8' 	
```

```java
import org.killbill.billing.client.api.gen.SubscriptionApi;
protected SubscriptionApi subscriptionApi;

UUID subscriptionId = UUID.fromString("1bb4b638-3886-4f73-90a5-89eb6d1bcf7f");

UUID tagDefinitionId = UUID.fromString("353752dd-9041-4450-b782-a8bb03a923c8");

subscriptionApi.deleteSubscriptionTags(subscriptionId, 
                                       ImmutableList.<UUID>of(tagDefinitionId), 
                                       requestOptions);
```

```ruby
tag_name = 'foo'

subscription.remove_tag(tag_name,
                        user,
                        reason,
                        comment,
                        options)
```

```python
subscriptionApi = killbill.api.SubscriptionApi()
subscription_id = 'f5bb14ed-c6e8-4895-8d4e-34422e12cdfa'
tag = ["353752dd-9041-4450-b782-a8bb03a923c8"]

subscriptionApi.delete_subscription_tags(subscription_id,
                                         created_by,
                                         api_key,
                                         api_secret,
                                         tag_def=tag)
```

> Example Response:

```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 204 No Content
< Content-Type: application/json
< Content-Length: 0
```

```java
no content
```
```ruby
no content
```
```python
no content
```

**Query Parameters**

| Name | Type | Required | Description |
| ---- | -----| -------- | ---- | ------------
| **tagList** | string | true |  list of tags that you want to remove it |

**Response**

A `204` http status without content.


## Audit Logs

### Retrieve subscription audit logs with history by subscription id

**HTTP Request** 

`GET http://127.0.0.1:8080/1.0/kb/subscriptions/{subscriptionId}/auditLogsWithHistory`

> Example Request:

```shell
curl -v \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Accept: application/json" \
    "http://localhost:8080/1.0/kb/subscriptions/70b6856e-6938-495f-9ae9-0a8ec0571c37/auditLogsWithHistory"
```
> Example Response:

```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 200 OK
< Content-Type: application/json

[
  {
    "changeType": "INSERT",
    "changeDate": "2019-02-22T22:38:10.000Z",
    "objectType": "SUBSCRIPTION",
    "objectId": "70b6856e-6938-495f-9ae9-0a8ec0571c37",
    "changedBy": "admin",
    "reasonCode": null,
    "comments": null,
    "userToken": "1f03e074-dea1-45c5-aee3-c9464886f476",
    "history": {
      "id": null,
      "createdDate": "2019-02-22T22:38:10.000Z",
      "updatedDate": "2019-02-22T22:38:10.000Z",
      "recordId": 465,
      "accountRecordId": 10,
      "tenantRecordId": 1,
      "bundleId": "d1b329c7-7dcf-466c-aaca-47bff304dab0",
      "category": "BASE",
      "startDate": "2019-02-22T22:38:10.000Z",
      "bundleStartDate": "2019-02-22T22:38:10.000Z",
      "chargedThroughDate": null,
      "migrated": false,
      "tableName": "SUBSCRIPTIONS",
      "historyTableName": "SUBSCRIPTION_HISTORY"
    }
  },
  {
    "changeType": "UPDATE",
    "changeDate": "2019-02-22T22:38:10.000Z",
    "objectType": "SUBSCRIPTION",
    "objectId": "70b6856e-6938-495f-9ae9-0a8ec0571c37",
    "changedBy": "SubscriptionBaseTransition",
    "reasonCode": null,
    "comments": null,
    "userToken": "1f03e074-dea1-45c5-aee3-c9464886f476",
    "history": {
      "id": null,
      "createdDate": "2019-02-22T22:38:10.000Z",
      "updatedDate": "2019-02-22T22:38:10.000Z",
      "recordId": 465,
      "accountRecordId": 10,
      "tenantRecordId": 1,
      "bundleId": "d1b329c7-7dcf-466c-aaca-47bff304dab0",
      "category": "BASE",
      "startDate": "2019-02-22T22:38:10.000Z",
      "bundleStartDate": "2019-02-22T22:38:10.000Z",
      "chargedThroughDate": "2019-03-22T23:35:14.000Z",
      "migrated": false,
      "tableName": "SUBSCRIPTIONS",
      "historyTableName": "SUBSCRIPTION_HISTORY"
    }
  }
]

```

**Query Parameters**

None.

**Returns**
    
Returns a list of subscription audit logs with history.



### Retrieve subscription event audit logs with history by subscription event id

**HTTP Request** 

`GET http://127.0.0.1:8080/1.0/kb/subscriptions/events/{subscriptionEventId}/auditLogsWithHistory`

The id of subscription event is the one comes from the [timeline api](#account-retrieve-account-timeline).


> Example Request:

```shell
curl  \
    -u admin:password \
    -H "X-Killbill-ApiKey: bob" \
    -H "X-Killbill-ApiSecret: lazar" \
    -H "Accept: application/json" \
    "http://localhost:8080/1.0/kb/subscriptions/events/9a1c85d5-beba-40c8-9268-f73e09c24007/auditLogsWithHistory"
```
> Example Response:


```shell
# Subset of headers returned when specifying -v curl option
< HTTP/1.1 200 OK
< Content-Type: application/json

[
  {
    "changeType": "INSERT",
    "changeDate": "2019-02-22T22:38:10.000Z",
    "objectType": "SUBSCRIPTION_EVENT",
    "objectId": "9a1c85d5-beba-40c8-9268-f73e09c24007",
    "changedBy": "admin",
    "reasonCode": null,
    "comments": null,
    "userToken": "1f03e074-dea1-45c5-aee3-c9464886f476",
    "history": {
      "id": null,
      "createdDate": "2019-02-22T22:38:10.000Z",
      "updatedDate": "2019-02-22T22:38:10.000Z",
      "recordId": 1358,
      "accountRecordId": 10,
      "tenantRecordId": 1,
      "totalOrdering": 0,
      "eventType": "API_USER",
      "userType": "CREATE",
      "effectiveDate": "2019-02-22T22:38:10.000Z",
      "subscriptionId": "70b6856e-6938-495f-9ae9-0a8ec0571c37",
      "planName": "foo-monthly",
      "phaseName": "foo-monthly-evergreen",
      "priceListName": "DEFAULT",
      "billingCycleDayLocal": 0,
      "isActive": true,
      "tableName": "SUBSCRIPTION_EVENTS",
      "historyTableName": "SUBSCRIPTION_EVENT_HISTORY",
      "active": true
    }
  }
]
```
**Query Parameters**

None.

**Returns**
    
Returns a list of subscription event audit logs with history.

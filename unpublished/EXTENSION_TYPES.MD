# Wix App Extensions — Complete Reference

Generated from `@wix/app-extensions` package.

---

## Public Extensions (Builder Functions)

These are the exported functions you use in `src/extensions.ts`.

### Site Extensions

| Extension | Function | Purpose |
|---|---|---|
| Widget | `widget()` | An iframe displayed as a draggable widget on a user's site. Users can resize, move, or pin it. |
| Page | `page()` | An iframe displayed as a full page on a user's site (appears in site navigation). |
| Worker | `worker()` | An invisible iframe added to every page on the site (max 1). Used for silent activity tracking. |
| Embedded Script | `embeddedScript()` | Injects a third-party script into the site's HTML. Used for analytics, tracking, chat, etc. |
| Widget Plugin | `widgetPlugin()` | A widget that renders inside a predefined slot in a Wix app's page (e.g. Bookings slot). |
| Unified Page | `unifiedPage()` | A site page that displays selected widgets — the newer unified version of the page extension. |
| Custom Element Widget | `customElementWidget()` | A draggable widget built as a self-hosted web component (custom element). |
| Editor Addon | `editorAddon()` | Adds functionality to the Wix editor itself (tool panel, editor script). |

### Dashboard Extensions

| Extension | Function | Purpose |
|---|---|---|
| Back Office Page | `backOfficePage()` | A page embedded in the Wix Dashboard. Supports iframe URL or Wix CLI route path. |
| Back Office Modal | `backOfficeModal()` | A reusable modal/popup for the Wix Dashboard, usable across pages and apps. |
| Dashboard | `dashboard()` | Opens an iframe in the Dashboard, or adds an external link to your platform. |

### Backend / Data Extensions

| Extension | Function | Purpose |
|---|---|---|
| Data Component | `dataComponent()` | Adds CMS data collections to a site when the app is installed. |
| Data Extensions | `dataExtensions()` | Extends an existing Wix entity (contacts, products, etc.) with custom fields. |
| External Database Provider | `externalDatabaseProvider()` | Connects an external database to Wix CMS via SPI. |
| Multilingual Translation Schema | `multilingualTranslationSchema()` | Defines a schema for translatable app content in the Wix Multilingual system. |
| Multilingual Translation Schema Group | `multilingualTranslationSchemaGroup()` | Groups multiple translation schemas together. |

### eCommerce Service Plugins (SPIs)

| Extension | Function | Purpose |
|---|---|---|
| Ecom Shipping Rates | `ecomShippingRates()` | Provides custom shipping rate calculations to cart and checkout. |
| Ecom Additional Fees | `ecomAdditionalFees()` | Adds custom fees (e.g. handling charges) to cart and checkout. |
| Ecom Catalog | `ecomCatalog()` | Makes your app a Wix catalog provider — integrates an external product repository. |
| Ecom Gift Cards Provider | `giftCardsProvider()` | Implements gift card balance retrieval, redemption, and voiding. |
| Ecom Validations | `ecomValidations()` | Validates cart/checkout items based on custom logic; can block checkout. |
| Ecom Payment Settings | `ecomPaymentSettings()` | Integrates with Wix's payment process (e.g. 3DS settings). |
| Ecom Inventory | `ecomInventory()` | Integrates an external inventory system with the Wix eCommerce platform. |

### Other Service Plugins (SPIs)

| Extension | Function | Purpose |
|---|---|---|
| Bookings Pricing Provider | `bookingsPricingProvider()` | Provides custom pricing logic for Wix Bookings. |
| Staff Sorting Provider | `staffSortingProvider()` | Provides a custom sort order for staff members in Wix Bookings. |
| Stores Product Restrictions | `storesProductRestrictions()` | Restricts editing of products managed by third-party fulfillment apps (dropshipping, POD). |
| Premium Custom Charges | `premiumCustomCharges()` | Applies custom billing charges based on your app's service usage. |
| SEO Keywords Suggestions | `seoKeywordsSuggestions()` | Provides SEO keyword suggestions to site collaborators with quota reporting. |
| Realtime Permissions Provider | `realtimePermissionsProvider()` | Controls who can receive messages on your app's Wix Realtime channels. |

---

## Internal Component Types (`ComponentType` enum)

These are internal Wix platform classification types. They are NOT exported as builder functions and cannot be used directly in app code, but they appear in the `ComponentType` enum.

| Enum Value | Notes |
|---|---|
| `NONE` | Default/unset value |
| `STUDIO` | Wix Studio component |
| `PLATFORM` | Platform-level component |
| `WORKER` | Invisible iframe worker (maps to `worker()`) |
| `DASHBOARD` | Dashboard iframe (maps to `dashboard()`) |
| `WIDGET` | Site widget iframe (maps to `widget()`) |
| `PAGE` | Site page iframe (maps to `page()`) |
| `DASHBOARD_PLATFORM` | Internal dashboard platform component |
| `STUDIO_WIDGET` | Wix Studio widget |
| `EMBEDDED_SCRIPT` | Embedded script (maps to `embeddedScript()`) |
| `EXTENSION` | Generic extension container |
| `SNIPPET_SOLUTION` | Internal snippet/solution type |
| `DATA_COMPONENT` | Data collection (maps to `dataComponent()`) |
| `WEB` | Generic web component |
| `DC_CONFIG` | Data collection configuration |
| `WIDGET_OUT_OF_IFRAME` | Widget rendered without iframe (out-of-iframe) |
| `PAGE_OUT_OF_IFRAME` | Page rendered without iframe (out-of-iframe) |
| `STATIC_FILE` | Static file asset |
| `APP_CONFIG` | App-level configuration component |
| `MULTIPLE_DASHBOARDS` | Multiple dashboard pages container |
| `PAYMENTS_GATEWAY` | Payment gateway integration |
| `CODE_PACKAGE` | Code package/library |
| `AUTOMATION_TRIGGER` | Wix Automations trigger |
| `INVOICES_ACTIONS` | Invoice action extension |
| `DASHBOARD_APPLICATION` | Dashboard application container |
| `CONTACT_LABELS` | CRM contact label extension |
| `WIDGET_PLUGIN` | Site plugin widget (maps to `widgetPlugin()`) |
| `CROSS_SELL` | Cross-sell product integration |
| `LOCAL_DELIVERY` | Local delivery integration |
| `PAYMENT_PROVIDER` | Payment provider integration |
| `ECOM_MEMBERSHIPS` | eCommerce memberships integration |
| `ECOM_LINE_ITEMS_ENRICHER` | Enriches eCommerce line items |
| `ECOM_SHIPPING_RATES` | Shipping rates SPI (maps to `ecomShippingRates()`) |
| `SHIPPING_LABEL_CARRIER` | Shipping label carrier integration |
| `RESTAURANTS_POS` | Restaurant POS integration |
| `FICTIONAL_SHIPPING_PROVIDER` | Internal test/fictional shipping provider |
| `ALERT_ENRICHER` | Alert enrichment extension |
| `DATA_EXTENSIONS` | Data extensions SPI (maps to `dataExtensions()`) |
| `GENERIC_HOOKS` | Generic lifecycle hooks |
| `AUTOMATIONS_ACTION_PROVIDER` | Wix Automations custom action provider |
| `ECOM_CATALOG` | eCommerce catalog SPI (maps to `ecomCatalog()`) |
| `BACK_OFFICE_EXTENSION_CONTAINER` | Container for back-office extensions |
| `BACK_OFFICE_EXTENSION` | Generic back-office extension |
| `AUTOMATIONS_TRIGGER_PROVIDER` | Wix Automations custom trigger provider |
| `IDENTITY_PRE_REGISTRATION` | Pre-registration identity hook |
| `PREMIUM_PRODUCTS_PATHS` | Premium product path configuration |
| `ECOM_CUSTOM_SCOPE` | Custom discount scope for eCommerce |
| `GIFT_CARDS_PROVIDER` | Gift card SPI (maps to `giftCardsProvider()`) |
| `DEALER_EXTERNAL_FILTER_PROVIDER` | Dealer external filter provider |
| `ECOM_DROPSHIPPING_PROVIDER` | Dropshipping provider integration |
| `INVOICES_PROVIDER` | Invoice provider SPI |
| `SEO_KEYWORDS_SUGGESTIONS` | SEO keywords SPI (maps to `seoKeywordsSuggestions()`) |
| `ECOM_DISCOUNTS_TRIGGER` | Custom discount trigger for eCommerce |
| `MULTILINGUAL_CONTENT_PROVIDER` | Multilingual content provider |
| `APPLICATION_AUTOMATION` | Application-level automation |
| `BACK_OFFICE_SIDEBAR_CATEGORY` | Dashboard sidebar category grouping |
| `BACK_OFFICE_PAGE` | Dashboard page (maps to `backOfficePage()`) |
| `ECOM_ADDITIONAL_FEES` | Additional fees SPI (maps to `ecomAdditionalFees()`) |
| `PING_USER_NOTIFICATION` | Push/ping user notification |
| `ECOM_RECOMMENDATIONS_PROVIDER` | eCommerce product recommendations provider |
| `BOOKINGS_PRICING_PROVIDER` | Bookings pricing SPI (maps to `bookingsPricingProvider()`) |
| `IDENTITY_AUTHENTICATOR` | Custom identity authenticator |
| `IDENTITY_IDP_CONNECTOR` | Identity provider connector |
| `ITEMS_SELECTION_PROVIDER` | Items selection provider |
| `PORTFOLIO_SYNCED_PROJECTS_PROVIDER` | Portfolio synced projects provider |
| `COMMUNICATION_CHANNEL` | Communication channel extension |
| `IDENTITY_POST_LOGIN` | Post-login identity hook |
| `BACK_OFFICE_WIDGET` | Dashboard widget extension |
| `SOCIAL_MARKETING_DESIGN` | Social marketing design integration |
| `FORMS_SUBMISSIONS_PROVIDER` | Forms submissions provider |
| `WIX_OFFERING` | Wix offering/product integration |
| `DEV_CENTER_TESTING_COMPONENT` | Dev Center testing/QA component |
| `COMPONENTS_VALIDATOR_PROVIDER` | Component validation provider |
| `COMPONENTS_TRANSLATIONS_ADDITIONAL_FIELDS_PROVIDER` | Additional fields for component translations |
| `FORMS_SCHEMA_PROVIDER` | Forms schema provider |
| `BOOKINGS_EXTERNAL_CALENDAR_PROVIDER` | External calendar provider for Bookings |
| `ECOM_DEFAULT_TAXATION_CATEGORY` | Default taxation category for eCommerce |
| `VIEWER_DYNAMIC_SITE_STRUCTURE_PROVIDER` | Dynamic site structure provider for viewer |
| `PING_UOU_NOTIFICATION` | User-to-user push notification |
| `HEADLESS_OAUTH` | Headless OAuth integration |
| `ECOM_TAX_CALCULATOR_SPI` | Tax calculation SPI for eCommerce |
| `COMMENTS_MODERATION_PROVIDER` | Comments moderation provider |
| `GRID_APP_FILES_TRANSFORMER` | Grid app files transformer |
| `BENEFIT_PROGRAMS_POLICY_PROVIDER` | Benefit programs policy provider |
| `PREMIUM_CUSTOM_CHARGES` | Custom charges SPI (maps to `premiumCustomCharges()`) |
| `ECOM_VALIDATIONS` | Checkout validations SPI (maps to `ecomValidations()`) |
| `COMPONENT_REFERENCE_DATA_PROVIDER` | Component reference data provider |
| `WIX_REVIEWS_PRODUCT_CATALOG` | Wix Reviews product catalog integration |
| `SOCIAL_MARKETING_DESIGNS_PROVIDER` | Social marketing designs provider |
| `GOOGLE_BUSINESS_PROFILE_FEATURE_PROVIDER` | Google Business Profile feature provider |
| `COMMENTS_FILTER_PROVIDER` | Comments filter provider |
| `BILLING_TAX_ID_VALIDATOR` | Billing tax ID validator |
| `PING_SETTINGS_GROUP` | Push notification settings group |
| `FORMS_SPAM_SUBMISSIONS_PROVIDER` | Forms spam submissions provider |
| `EDITOR_ADDON` | Editor addon (maps to `editorAddon()`) |
| `EXTERNAL_DATABASE_PROVIDER` | External database SPI (maps to `externalDatabaseProvider()`) |
| `ECOM_PAYMENT_SETTINGS` | Payment settings SPI (maps to `ecomPaymentSettings()`) |
| `NOTIFICATION_TOPIC` | Notification topic definition |
| `NOTIFICATION_PREFERENCES_FILTER_PROVIDER` | Notification preferences filter provider |
| `BOOKINGS_RESOURCE_TYPES_PROVIDER` | Bookings resource types provider |
| `PRICING_PLANS_FORM_CONFIGURATION` | Pricing Plans form configuration |
| `USER_NOTIFICATION` | User notification extension |
| `CONTACT_NOTIFICATION` | Contact notification extension |
| `UNIFIED_PAGE` | Unified site page (maps to `unifiedPage()`) |
| `AVAILABILITY_TIME_SLOTS_CONFIGURATION_PROVIDER` | Availability time slots configuration |
| `PROPOSAL_EDITOR_PROVIDER` | Proposal editor provider |
| `CUSTOM_TABLE_RESERVATIONS_PROVIDER` | Custom table reservations provider |
| `COMMENTS_CONTEXT_PROVIDER` | Comments context provider |
| `FORMS_SPAM_SUBMISSION_REPORTS_PROVIDER` | Forms spam submission reports provider |
| `AUTOMATIONS_VELO_ACTION_PROVIDER` | Wix Automations Velo action provider |
| `CALENDAR_EVENT_TYPE_PROVIDER` | Calendar event type provider |
| `RESERVED` | Reserved (previously `SERVICE_AVAILABILITY_POLICY_PROVIDER`) |
| `SMS_ACTION_MESSAGE` | SMS action message extension |
| `BOOKING_POLICY_PROVIDER` | Booking policy provider |
| `MULTI_SERVICE_BOOKING_POLICY_PROVIDER` | Multi-service booking policy provider |
| `AI_ASSISTANT` | AI assistant integration |
| `FORMS_SUBMISSIONS_EXTENSION_PROVIDER` | Forms submissions extension provider |
| `MULTILINGUAL_TRANSLATION_SCHEMA` | Translation schema (maps to `multilingualTranslationSchema()`) |
| `TAX_CALCULATION_PROVIDER` | Tax calculation provider |
| `TAX_GROUPS_PROVIDER` | Tax groups provider |
| `BACK_OFFICE_MODAL` | Dashboard modal (maps to `backOfficeModal()`) |
| `DEPLOYMENT_PIPELINE_PROVIDER` | Deployment pipeline provider |
| `CUSTOM_ELEMENT_WIDGET` | Custom element widget (maps to `customElementWidget()`) |
| `BACK_OFFICE_EXTENSION_WIDGET` | Back-office extension widget |
| `BACK_OFFICE_EXTENSION_MENU_ITEM` | Back-office extension menu item |
| `FORM_TEMPLATE` | Form template extension |
| `NOTIFICATION_CONTENT` | Notification content extension |
| `BROADCAST_LIST` | Broadcast list extension |
| `PARTNERS_PAYOUTS` | Partners payouts integration |
| `WIX_REVIEWS_ENTITY_CATALOG` | Wix Reviews entity catalog |
| `VELO_PUBLISH_PIPELINE_TASK_PROVIDER` | Velo publish pipeline task provider |
| `FUNCTIONS_SHOP_PRICE_PROVIDER` | Functions shop price provider |
| `FUNCTION` | Function extension |
| `ECOM_CHECKOUT_CONTENT` | eCommerce checkout content extension |
| `COMMUNICATION_CHANNEL_PROVIDER` | Communication channel provider |
| `WEBHOOK` | Webhook extension |
| `TOPOLOGY` | Topology configuration |
| `LOYALTY_CUSTOM_REWARDS` | Loyalty custom rewards extension |
| `FUNCTION_RECIPE` | Function recipe extension |
| `BACK_OFFICE_EXTERNAL_URL` | Back-office external URL link |
| `IDENTITY_FACTOR` | Identity factor extension |
| `ECOM_DISCOUNTS` | eCommerce discounts extension |
| `VELO_CUSTOM_CSS` | Velo custom CSS extension |
| `DEALER_ADAPTIVE_COMPONENT_PROVIDER` | Dealer adaptive component provider |
| `AI_ASSISTANT_ACTION` | AI assistant action extension |
| `ECOM_INVENTORY` | Inventory SPI (maps to `ecomInventory()`) |
| `MONITORING` | Monitoring extension |
| `PAPI_PROVIDER` | PAPI (Public API) provider |
| `UNIFIED_LIGHTBOX` | Unified lightbox extension |
| `FORMS_EXTENSION_PROVIDER` | Forms extension provider |
| `MULTILINGUAL_TRANSLATION_SCHEMA_GROUP` | Translation schema group (maps to `multilingualTranslationSchemaGroup()`) |
| `PANEL` | Editor panel extension |
| `PREMIUM_PURCHASE_VALIDATIONS` | Premium purchase validations |
| `BROWSER_STORAGE` | Browser storage extension |
| `SDK_DEFINITION` | SDK definition extension |
| `SITE_WIDGET_SLOT` | Site widget slot definition |
| `CALENDAR_EVENT_VALIDATION_PROVIDER` | Calendar event validation provider |
| `CLIENT_SIDE_SERVICE` | Client-side service extension |
| `BILLING_SETTINGS` | Billing settings extension |
| `PATTERNS_WIZARD` | Patterns wizard extension |
| `APPLICATION_PROFILE` | Application profile extension |
| `TEXT_TO_SPEECH_ACTION_MESSAGE` | Text-to-speech action message |
| `AUDIENCE_PROVIDER` | Audience provider extension |
| `PRICING_PLANS_PRICE` | Pricing Plans price extension |
| `PRICING_PLAN_START_DATE_LIMITS` | Pricing Plans start date limits |
| `PRICING_PLAN_START_DATE_RULES` | Pricing Plans start date rules |
| `EVENTS_TICKET_RESERVATIONS` | Events ticket reservations extension |
| `PAYMENTS_DISPUTE_SERVICE_PLUGIN` | Payments dispute service plugin |
| `PRICING_PLANS_FEES` | Pricing Plans fees extension |
| `EDITOR_REACT_COMPONENT` | Editor React component extension |
| `SUPPLIERS_HUB_MARKETPLACE` | Suppliers hub marketplace integration |
| `FORM_SCHEMA_DYNAMIC_VALUES` | Form schema dynamic values provider |
| `BLOG_PAYWALL_PROVIDER` | Blog paywall provider |
| `LOYALTY_CUSTOM_REWARDS_V2` | Loyalty custom rewards (v2) |
| `STORES_PRODUCT_RESTRICTIONS` | Stores product restrictions SPI (maps to `storesProductRestrictions()`) |
| `FORM_SUBMISSION_MODERATION` | Form submission moderation |
| `EVENTS_EVENT_BADGES` | Events event badges extension |
| `BILLING_OPERATION` | Billing operation extension |
| `BACK_OFFICE_CUSTOMIZATION` | Back-office customization extension |
| `COMPONENT_ENRICHER_PROVIDER` | Component enricher provider |
| `BACK_OFFICE_RESTRICTED_CUSTOMIZATION` | Restricted back-office customization |
| `EDITOR_APP_PREVIEWS_POC` | Editor app previews (POC) |
| `LEGENDS_PERSONA_CONFIGURATION` | Legends persona configuration |
| `WIX_HOSTING_APP_DEPLOYMENT_PROVIDER` | Wix hosting app deployment provider |
| `BACKEND_WORKER` | Backend worker extension |
| `EVENT_TIME_SLOTS_CONFIGURATION_PROVIDER` | Event time slots configuration provider |
| `WIX_HOSTING_APP_ENVIRONMENT_PROVIDER` | Wix hosting app environment provider |
| `SDK_EXPORTS` | SDK exports extension |
| `VIEWER_SERVICE` | Viewer service extension |
| `WIX_PAYMENTS_PROVIDER_ACCOUNT_SERVICE_PLUGIN` | Wix Payments provider account service plugin |
| `CONTACTS_SEGMENTS_V2_FILTER_PROVIDER` | Contacts segments v2 filter provider |
| `ANALYTICS_PRODUCT_CATALOG` | Analytics product catalog |
| `SITE_MIGRATION` | Site migration extension |
| `RESTAURANTS_POS_PROVIDER_SETTINGS_SERVICE_PLUGIN` | Restaurants POS provider settings service plugin |
| `WIX_VIBE_CODING_INSTRUCTIONS` | Wix Vibe coding instructions |
| `TAX_EXEMPT_GROUPS_PROVIDER` | Tax-exempt groups provider |
| `BOOKING_AUTOMATIONS_CONFIGURATION` | Booking automations configuration |
| `WIX_VIBE_COMPONENT` | Wix Vibe component |
| `WIXEL` | Wixel extension |
| `STAFF_SORTING_PROVIDER` | Staff sorting SPI (maps to `staffSortingProvider()`) |
| `FUNCTION_ECOM_DISCOUNT_TRIGGER_ELIGIBILITY_PROVIDER` | Function-based eCommerce discount trigger eligibility provider |
| `FUNCTIONS_ASSISTANT_TOOLS` | Functions assistant tools |
| `SITE_MAP_INFO_PROVIDER` | Site map info provider |
| `WIX_AI_GATEWAY_CALLBACK` | Wix AI gateway callback |
| `URL_MAPPER_PROVIDER` | URL mapper provider |
| `PROVIDER_SUPPRESSION_SERVICE_PLUGIN` | Provider suppression service plugin |
| `ECOM_FORMS` | eCommerce forms extension |
| `EDITOR_CONTEXT_PROVIDER` | Editor context provider |
| `EDITOR_FUNCTION_LIBRARY` | Editor function library |
| `BOOKINGS_PLATFORM_CONFIGURATION` | Bookings platform configuration |
| `BOOKINGS_VALIDATION_PROVIDER` | Bookings validation provider |
| `APP_ROUTER` | App router extension |
| `WIXEL_BINDING` | Wixel binding extension |
| `SHUB_MARKETPLACE_PROVIDER` | Shub marketplace provider |
| `BOOKING_ACTION_URLS_PROVIDER` | Booking action URLs provider |
| `CALENDAR_CONFERENCING_PROVIDER` | Calendar conferencing provider |
| `BASS_VALIDATE_CANCEL_ACTION` | BASS validate cancel action |
| `WIXEL_VIEWER_ROUTER_PROVIDER` | Wixel viewer router provider |
| `PREMIUM_ACTION` | Premium action extension |
| `AUTOMATION_TEMPLATE` | Automation template extension |
| `AGENT_MAKER_APPROVAL_PROVIDER` | Agent maker approval provider |
| `SERVICE_TAGS_CONFIGURATION` | Service tags configuration |
| `LOYALTY_POINTS_ADJUSTMENT` | Loyalty points adjustment extension |
| `LOYALTY_EARN_POINTS_LIMIT` | Loyalty earn points limit extension |
| `REALTIME_PERMISSIONS_PROVIDER` | Realtime permissions SPI (maps to `realtimePermissionsProvider()`) |
| `CALENDAR_SEARCH_PROVIDER` | Calendar search provider |
| `EDITOR_INSTALLATION` | Editor installation extension |
| `APP_ACTIONS` | App actions extension |
| `ROLLOUT_STEP_EXECUTOR` | Rollout step executor |
| `BOOKINGS_PLATFORM_PREMIUM_VALIDATION` | Bookings platform premium validation |
| `GLOBAL_CSS` | Global CSS extension |
| `CONNECTOR_CONFIG` | Connector configuration |
| `CHANNEL_DATA_PROVIDER` | Channel data provider |
| `OAUTH_CLIENT_CONFIGURATION` | OAuth client configuration |
| `EDITOR_EXPOSURE` | Editor exposure extension |

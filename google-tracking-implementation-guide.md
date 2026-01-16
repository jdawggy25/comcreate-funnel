# Google Tracking Implementation Guide for Local Service Landing Pages

## Complete Implementation Reference for Google Ads Campaigns

**Document Version:** 1.0
**Last Updated:** January 2026
**Audience:** Developers, Marketing Teams, Analytics Specialists

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Google Tag Manager Setup](#2-google-tag-manager-setup)
3. [Google Analytics 4 Configuration](#3-google-analytics-4-configuration)
4. [Google Ads Conversion Tracking](#4-google-ads-conversion-tracking)
5. [Phone Call Tracking](#5-phone-call-tracking)
6. [Form Submission Tracking](#6-form-submission-tracking)
7. [Remarketing Setup](#7-remarketing-setup)
8. [Quality Score Optimization](#8-quality-score-optimization)
9. [Testing and Validation](#9-testing-and-validation)
10. [Troubleshooting Guide](#10-troubleshooting-guide)

---

## 1. Architecture Overview

### Tracking Stack for Local Service Businesses

```
┌─────────────────────────────────────────────────────────────────┐
│                     LANDING PAGE                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    DATA LAYER                            │   │
│  │  - Page data, user interactions, form data, events       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                            │                                     │
│                            ▼                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              GOOGLE TAG MANAGER                          │   │
│  │  - Container manages all tags                            │   │
│  │  - Triggers fire based on events                         │   │
│  │  - Variables extract data layer values                   │   │
│  └─────────────────────────────────────────────────────────┘   │
│                            │                                     │
│              ┌─────────────┼─────────────┐                      │
│              ▼             ▼             ▼                      │
│      ┌───────────┐  ┌───────────┐  ┌───────────┐               │
│      │   GA4     │  │  Google   │  │ Remarketing│               │
│      │  Events   │  │   Ads     │  │    Tags    │               │
│      │           │  │Conversions│  │            │               │
│      └───────────┘  └───────────┘  └───────────┘               │
└─────────────────────────────────────────────────────────────────┘
```

### Key Tracking Events for Local Services

| Event Category | Specific Events | Business Value |
|---------------|-----------------|----------------|
| **Lead Generation** | Form submission, Quote request | Primary conversion |
| **Phone Engagement** | Click-to-call, Call completed | Primary conversion |
| **Engagement Signals** | Scroll depth, Time on page | Quality indicators |
| **Micro-conversions** | Service page views, Pricing views | Intent signals |

---

## 2. Google Tag Manager Setup

### 2.1 Container Creation and Installation

#### Step 1: Create GTM Account and Container

1. Navigate to [tagmanager.google.com](https://tagmanager.google.com)
2. Click "Create Account"
3. Enter account details:
   - Account Name: `[Business Name]`
   - Country: `[Your Country]`
4. Create Container:
   - Container Name: `[domain.com] - Landing Pages`
   - Target Platform: `Web`
5. Accept Terms of Service

#### Step 2: Install GTM Container Code

Place this code immediately after the opening `<head>` tag:

```html
<!-- Google Tag Manager -->
<script>
(function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
})(window,document,'script','dataLayer','GTM-XXXXXXX');
</script>
<!-- End Google Tag Manager -->
```

Place this code immediately after the opening `<body>` tag:

```html
<!-- Google Tag Manager (noscript) -->
<noscript><iframe src="https://www.googletagmanager.com/ns.html?id=GTM-XXXXXXX"
height="0" width="0" style="display:none;visibility:hidden"></iframe></noscript>
<!-- End Google Tag Manager (noscript) -->
```

**Replace `GTM-XXXXXXX` with your actual container ID.**

### 2.2 Data Layer Configuration

#### Base Data Layer Implementation

Place this BEFORE the GTM container code:

```html
<script>
window.dataLayer = window.dataLayer || [];

// Initialize data layer with page-level data
dataLayer.push({
  'event': 'page_data_ready',

  // Page Information
  'page': {
    'type': 'landing_page',           // landing_page, thank_you, service_page
    'service_category': 'plumbing',   // plumbing, hvac, electrical, roofing, etc.
    'service_type': 'emergency',      // emergency, scheduled, maintenance
    'location': {
      'city': 'Austin',
      'state': 'TX',
      'zip': '78701',
      'service_area': 'Central Austin'
    }
  },

  // Campaign Context (populated from URL parameters)
  'campaign': {
    'source': getUrlParameter('utm_source') || 'direct',
    'medium': getUrlParameter('utm_medium') || 'none',
    'campaign': getUrlParameter('utm_campaign') || 'none',
    'content': getUrlParameter('utm_content') || 'none',
    'keyword': getUrlParameter('utm_term') || 'none',
    'gclid': getUrlParameter('gclid') || 'none'
  },

  // Business Information
  'business': {
    'name': 'ABC Plumbing Services',
    'phone': '+15125551234',
    'formatted_phone': '(512) 555-1234'
  }
});

// Helper function to extract URL parameters
function getUrlParameter(name) {
  const urlParams = new URLSearchParams(window.location.search);
  return urlParams.get(name);
}

// Store GCLID in cookie for cross-session attribution
(function() {
  const gclid = getUrlParameter('gclid');
  if (gclid) {
    const expiryDate = new Date();
    expiryDate.setDate(expiryDate.getDate() + 90);
    document.cookie = 'gclid=' + gclid + ';expires=' + expiryDate.toUTCString() + ';path=/;SameSite=Lax';
  }
})();
</script>
```

#### Dynamic Data Layer Events

```javascript
// ============================================
// DATA LAYER EVENT FUNCTIONS
// Include in your main JavaScript file
// ============================================

const TrackingEvents = {

  // Form Interaction Tracking
  formStart: function(formId, formName) {
    dataLayer.push({
      'event': 'form_start',
      'form_id': formId,
      'form_name': formName,
      'form_location': this.getFormLocation(formId),
      'timestamp': new Date().toISOString()
    });
  },

  formFieldComplete: function(formId, fieldName, fieldType) {
    dataLayer.push({
      'event': 'form_field_complete',
      'form_id': formId,
      'field_name': fieldName,
      'field_type': fieldType
    });
  },

  formSubmit: function(formData) {
    dataLayer.push({
      'event': 'form_submission',
      'form_id': formData.formId,
      'form_name': formData.formName,
      'form_type': formData.formType,           // quote, contact, callback
      'service_requested': formData.service,
      'lead_value': formData.estimatedValue || 0,
      'urgency': formData.urgency || 'standard', // emergency, same_day, standard

      // Enhanced Conversions Data (hashed on server)
      'user_data': {
        'email': formData.email,
        'phone': formData.phone,
        'first_name': formData.firstName,
        'last_name': formData.lastName,
        'address': {
          'city': formData.city,
          'state': formData.state,
          'postal_code': formData.zip,
          'country': 'US'
        }
      }
    });
  },

  // Phone Click Tracking
  phoneClick: function(phoneNumber, clickLocation) {
    dataLayer.push({
      'event': 'phone_click',
      'phone_number': phoneNumber,
      'click_location': clickLocation,  // header, hero, footer, sticky_cta
      'page_scroll_depth': this.getCurrentScrollDepth(),
      'time_on_page': this.getTimeOnPage()
    });
  },

  // Scroll Depth Tracking
  scrollDepth: function(percentage) {
    dataLayer.push({
      'event': 'scroll_depth',
      'scroll_percentage': percentage,
      'scroll_threshold': percentage + '%'
    });
  },

  // Engagement Tracking
  elementVisible: function(elementId, elementName) {
    dataLayer.push({
      'event': 'element_visible',
      'element_id': elementId,
      'element_name': elementName
    });
  },

  ctaClick: function(ctaText, ctaLocation, ctaDestination) {
    dataLayer.push({
      'event': 'cta_click',
      'cta_text': ctaText,
      'cta_location': ctaLocation,
      'cta_destination': ctaDestination
    });
  },

  // Helper Methods
  getFormLocation: function(formId) {
    const form = document.getElementById(formId);
    if (!form) return 'unknown';
    const rect = form.getBoundingClientRect();
    const viewportHeight = window.innerHeight;
    if (rect.top < viewportHeight * 0.5) return 'above_fold';
    return 'below_fold';
  },

  getCurrentScrollDepth: function() {
    const scrollTop = window.pageYOffset || document.documentElement.scrollTop;
    const docHeight = document.documentElement.scrollHeight - window.innerHeight;
    return Math.round((scrollTop / docHeight) * 100);
  },

  getTimeOnPage: function() {
    return Math.round((Date.now() - window.pageLoadTime) / 1000);
  }
};

// Initialize page load time
window.pageLoadTime = Date.now();
```

### 2.3 GTM Variables Configuration

#### Built-in Variables to Enable

In GTM, go to Variables > Configure Built-in Variables and enable:

**Pages:**
- Page URL
- Page Hostname
- Page Path
- Referrer

**Clicks:**
- Click Element
- Click Classes
- Click ID
- Click Target
- Click URL
- Click Text

**Forms:**
- Form Element
- Form Classes
- Form ID
- Form Target
- Form URL
- Form Text

**Utilities:**
- Event
- Container ID
- Container Version
- Random Number
- HTML ID

#### Custom Data Layer Variables

Create these User-Defined Variables in GTM:

**Variable 1: DLV - Form ID**
```
Variable Type: Data Layer Variable
Data Layer Variable Name: form_id
Data Layer Version: Version 2
```

**Variable 2: DLV - Form Name**
```
Variable Type: Data Layer Variable
Data Layer Variable Name: form_name
Data Layer Version: Version 2
```

**Variable 3: DLV - Service Requested**
```
Variable Type: Data Layer Variable
Data Layer Variable Name: service_requested
Data Layer Version: Version 2
```

**Variable 4: DLV - Lead Value**
```
Variable Type: Data Layer Variable
Data Layer Variable Name: lead_value
Data Layer Version: Version 2
```

**Variable 5: DLV - Phone Number**
```
Variable Type: Data Layer Variable
Data Layer Variable Name: phone_number
Data Layer Version: Version 2
```

**Variable 6: DLV - Click Location**
```
Variable Type: Data Layer Variable
Data Layer Variable Name: click_location
Data Layer Version: Version 2
```

**Variable 7: DLV - Scroll Percentage**
```
Variable Type: Data Layer Variable
Data Layer Variable Name: scroll_percentage
Data Layer Version: Version 2
```

**Variable 8: DLV - User Email (for Enhanced Conversions)**
```
Variable Type: Data Layer Variable
Data Layer Variable Name: user_data.email
Data Layer Version: Version 2
```

**Variable 9: DLV - User Phone (for Enhanced Conversions)**
```
Variable Type: Data Layer Variable
Data Layer Variable Name: user_data.phone
Data Layer Version: Version 2
```

**Variable 10: DLV - GCLID**
```
Variable Type: Data Layer Variable
Data Layer Variable Name: campaign.gclid
Data Layer Version: Version 2
```

#### Lookup Table Variables

**Variable: Service Category Value**
```
Variable Type: Lookup Table
Input Variable: {{DLV - Service Requested}}

Lookup Table:
emergency_plumbing    -> 250
drain_cleaning        -> 150
water_heater          -> 500
hvac_repair          -> 300
ac_installation      -> 2500
electrical_repair    -> 200
roof_repair          -> 1000
Default Value        -> 100
```

### 2.4 GTM Triggers Configuration

#### Trigger 1: All Pages
```
Trigger Type: Page View
This trigger fires on: All Page Views
```

#### Trigger 2: Form Submission
```
Trigger Type: Custom Event
Event Name: form_submission
This trigger fires on: All Custom Events
```

#### Trigger 3: Phone Click
```
Trigger Type: Custom Event
Event Name: phone_click
This trigger fires on: All Custom Events
```

#### Trigger 4: Scroll Depth - 25%
```
Trigger Type: Scroll Depth
Vertical Scroll Depths: Percentages - 25
This trigger fires on: All Pages
```

#### Trigger 5: Scroll Depth - 50%
```
Trigger Type: Scroll Depth
Vertical Scroll Depths: Percentages - 50
This trigger fires on: All Pages
```

#### Trigger 6: Scroll Depth - 75%
```
Trigger Type: Scroll Depth
Vertical Scroll Depths: Percentages - 75
This trigger fires on: All Pages
```

#### Trigger 7: Scroll Depth - 90%
```
Trigger Type: Scroll Depth
Vertical Scroll Depths: Percentages - 90
This trigger fires on: All Pages
```

#### Trigger 8: Thank You Page
```
Trigger Type: Page View
This trigger fires on: Some Page Views
Condition: Page Path contains thank-you OR Page Path contains confirmation
```

#### Trigger 9: Click to Call (Link Click)
```
Trigger Type: Click - Just Links
This trigger fires on: Some Link Clicks
Condition: Click URL starts with tel:
```

#### Trigger 10: Form Start (First Interaction)
```
Trigger Type: Custom Event
Event Name: form_start
This trigger fires on: All Custom Events
```

#### Trigger 11: Element Visibility - Form
```
Trigger Type: Element Visibility
Selection Method: CSS Selector
Element Selector: form[data-track="lead-form"]
When to fire: Once per page
Minimum Percent Visible: 50
```

---

## 3. Google Analytics 4 Configuration

### 3.1 GA4 Property Setup

#### Step 1: Create GA4 Property

1. Go to [analytics.google.com](https://analytics.google.com)
2. Admin > Create Property
3. Property Setup:
   - Property Name: `[Business Name] - Landing Pages`
   - Reporting Time Zone: `[Your Time Zone]`
   - Currency: `USD`

#### Step 2: Create Web Data Stream

1. Admin > Data Streams > Add Stream > Web
2. Configure:
   - Website URL: `https://yourdomain.com`
   - Stream Name: `Landing Pages - Production`
3. Copy the Measurement ID (G-XXXXXXXXXX)

#### Step 3: Enhanced Measurement Settings

In Data Stream settings, configure Enhanced Measurement:

| Feature | Status | Notes |
|---------|--------|-------|
| Page views | ON | Automatic page tracking |
| Scrolls | OFF | We'll track custom thresholds |
| Outbound clicks | ON | Track external link clicks |
| Site search | ON | Configure search parameter: q, s, search |
| Video engagement | ON | For embedded videos |
| File downloads | ON | Track PDF/doc downloads |
| Form interactions | OFF | We'll track custom events |

### 3.2 GA4 Tag Configuration in GTM

#### Tag 1: GA4 Configuration Tag

```
Tag Type: Google Analytics: GA4 Configuration
Measurement ID: G-XXXXXXXXXX

Fields to Set:
  debug_mode: true (remove in production)
  send_page_view: true

User Properties:
  traffic_source: {{DLV - Campaign Source}}
  service_area: {{DLV - Service Area}}
  landing_page_type: {{DLV - Page Type}}

Triggering: All Pages
```

#### Tag 2: GA4 Event - Form Submission

```
Tag Type: Google Analytics: GA4 Event
Configuration Tag: GA4 Configuration Tag
Event Name: generate_lead

Event Parameters:
  form_id: {{DLV - Form ID}}
  form_name: {{DLV - Form Name}}
  service_requested: {{DLV - Service Requested}}
  lead_value: {{DLV - Lead Value}}
  currency: USD
  form_location: {{DLV - Form Location}}

User Properties:
  lead_status: submitted

Triggering: Form Submission
```

#### Tag 3: GA4 Event - Phone Click

```
Tag Type: Google Analytics: GA4 Event
Configuration Tag: GA4 Configuration Tag
Event Name: click_to_call

Event Parameters:
  phone_number: {{DLV - Phone Number}}
  click_location: {{DLV - Click Location}}
  link_url: {{Click URL}}

Triggering: Phone Click, Click to Call (Link Click)
```

#### Tag 4: GA4 Event - Scroll Depth

```
Tag Type: Google Analytics: GA4 Event
Configuration Tag: GA4 Configuration Tag
Event Name: scroll

Event Parameters:
  percent_scrolled: {{Scroll Depth Threshold}}

Triggering:
  - Scroll Depth - 25%
  - Scroll Depth - 50%
  - Scroll Depth - 75%
  - Scroll Depth - 90%
```

#### Tag 5: GA4 Event - Form Start

```
Tag Type: Google Analytics: GA4 Event
Configuration Tag: GA4 Configuration Tag
Event Name: form_start

Event Parameters:
  form_id: {{DLV - Form ID}}
  form_name: {{DLV - Form Name}}

Triggering: Form Start
```

#### Tag 6: GA4 Event - CTA Click

```
Tag Type: Google Analytics: GA4 Event
Configuration Tag: GA4 Configuration Tag
Event Name: cta_click

Event Parameters:
  cta_text: {{Click Text}}
  cta_location: {{DLV - CTA Location}}
  link_url: {{Click URL}}

Triggering: Custom Event (event name: cta_click)
```

### 3.3 GA4 Conversion Events Configuration

#### Mark Events as Conversions in GA4

1. Go to Admin > Events
2. Mark as Conversions:

| Event Name | Conversion Value | Counting |
|------------|------------------|----------|
| generate_lead | Use event parameter (lead_value) | Once per session |
| click_to_call | $50 default | Once per session |
| form_start | $0 (micro-conversion) | Once per session |

#### Create Conversion Value Rules

In GA4 Admin > Events > Modify Event:

**High-Value Lead Identification:**
```
Event Name: generate_lead
Condition: service_requested equals emergency_plumbing
Modify Parameters:
  value: 300
  priority: high
```

### 3.4 GA4 Audiences for Analysis

#### Audience 1: Engaged Visitors
```
Include users when:
  scroll percent_scrolled >= 75
  OR session_duration > 120 seconds

Membership Duration: 30 days
```

#### Audience 2: Form Abandoners
```
Include users when:
  form_start triggered

Exclude users when:
  generate_lead triggered

Membership Duration: 14 days
```

#### Audience 3: Phone Leads
```
Include users when:
  click_to_call triggered

Membership Duration: 30 days
```

#### Audience 4: High-Intent Visitors
```
Include users when:
  scroll percent_scrolled >= 90
  AND page_view count >= 2

Membership Duration: 7 days
```

---

## 4. Google Ads Conversion Tracking

### 4.1 Conversion Actions Setup

#### Step 1: Create Conversion Actions in Google Ads

Navigate to: Tools & Settings > Measurement > Conversions > New Conversion Action

**Conversion Action 1: Form Submission (Primary)**

```
Category: Submit lead form
Conversion name: Lead Form Submission
Value: Use different values for each conversion
Default value: $100
Count: One (per click)
Click-through conversion window: 30 days
View-through conversion window: 1 day
Attribution model: Data-driven (or Time decay)
Include in "Conversions": Yes
```

**Conversion Action 2: Phone Call Click (Primary)**

```
Category: Phone call leads
Conversion name: Phone Call Click
Value: $50
Count: One (per click)
Click-through conversion window: 30 days
View-through conversion window: 1 day
Attribution model: Data-driven
Include in "Conversions": Yes
```

**Conversion Action 3: Phone Call from Ads (Primary)**

```
Category: Phone calls
Source: Calls from ads using call extensions or call-only ads
Conversion name: Phone Call from Ad
Call length: 60 seconds
Value: $75
Count: One (per click)
Include in "Conversions": Yes
```

**Conversion Action 4: Phone Call to Website (Primary)**

```
Category: Phone calls
Source: Calls to a phone number on your website
Conversion name: Website Phone Call
Call length: 60 seconds
Value: $75
Count: One (per click)
Include in "Conversions": Yes
```

**Conversion Action 5: Form Start (Secondary/Observation)**

```
Category: Other
Conversion name: Form Interaction Start
Value: $0
Count: One (per click)
Click-through conversion window: 7 days
Attribution model: Last click
Include in "Conversions": No (observation only)
```

#### Step 2: Get Conversion IDs and Labels

After creating each conversion action, click into it to find:
- Conversion ID (numeric): `AW-XXXXXXXXX`
- Conversion Label: `XXXXXXXXXXXXXXXXXXX`

### 4.2 GTM Tags for Google Ads Conversions

#### Tag 1: Google Ads Conversion - Form Submission

```
Tag Type: Google Ads Conversion Tracking
Conversion ID: AW-XXXXXXXXX
Conversion Label: XXXXXXXXXXXXXXXXXXX
Conversion Value: {{DLV - Lead Value}}
Currency Code: USD
Transaction ID: {{DLV - Form ID}}_{{Random Number}}
Conversion Linker: Enabled

Triggering: Form Submission
```

#### Tag 2: Google Ads Conversion - Phone Click

```
Tag Type: Google Ads Conversion Tracking
Conversion ID: AW-XXXXXXXXX
Conversion Label: YYYYYYYYYYYYYYYYYYY
Conversion Value: 50
Currency Code: USD
Transaction ID: phone_{{Random Number}}

Triggering: Phone Click, Click to Call (Link Click)
```

#### Tag 3: Google Ads Conversion Linker

```
Tag Type: Conversion Linker
Enable linking on all page URLs: Checked
Enable cross-domain linking: Unchecked (unless needed)

Triggering: All Pages
```

### 4.3 Enhanced Conversions Setup

#### Step 1: Enable in Google Ads

1. Go to Tools & Settings > Measurement > Conversions
2. Click on your conversion action
3. Expand "Enhanced conversions"
4. Turn on "Enhanced conversions for leads"
5. Choose "Google Tag Manager"

#### Step 2: Configure Enhanced Conversions Tag in GTM

```
Tag Type: Google Ads Conversion Tracking

Conversion ID: AW-XXXXXXXXX
Conversion Label: XXXXXXXXXXXXXXXXXXX

Include user-provided data from your website: Checked

User-provided data:
  Data source: Manual configuration

  Email: {{DLV - User Email}}
  Phone: {{DLV - User Phone}}

  Address:
    First Name: {{DLV - User First Name}}
    Last Name: {{DLV - User Last Name}}
    Street: {{DLV - User Street}}
    City: {{DLV - User City}}
    Region: {{DLV - User State}}
    Postal Code: {{DLV - User Zip}}
    Country: US

Triggering: Form Submission
```

#### Enhanced Conversions Data Layer Implementation

```javascript
// On form submission, push enhanced conversion data
function handleFormSubmit(formElement) {
  const formData = new FormData(formElement);

  dataLayer.push({
    'event': 'form_submission',
    'form_id': formElement.id,
    'form_name': formElement.dataset.formName,
    'form_type': formElement.dataset.formType,
    'service_requested': formData.get('service'),
    'lead_value': calculateLeadValue(formData.get('service')),

    // Enhanced Conversions - User Data
    // Google will hash this data automatically
    'user_data': {
      'email': formData.get('email'),
      'phone_number': normalizePhone(formData.get('phone')),
      'address': {
        'first_name': formData.get('firstName'),
        'last_name': formData.get('lastName'),
        'street': formData.get('address'),
        'city': formData.get('city'),
        'region': formData.get('state'),
        'postal_code': formData.get('zip'),
        'country': 'US'
      }
    }
  });
}

// Phone normalization function
function normalizePhone(phone) {
  if (!phone) return '';
  // Remove all non-numeric characters
  const cleaned = phone.replace(/\D/g, '');
  // Add country code if not present
  if (cleaned.length === 10) {
    return '+1' + cleaned;
  }
  return '+' + cleaned;
}

// Lead value calculation
function calculateLeadValue(service) {
  const values = {
    'emergency_plumbing': 300,
    'drain_cleaning': 150,
    'water_heater': 500,
    'hvac_repair': 350,
    'ac_installation': 2500,
    'electrical': 200,
    'roofing': 1000
  };
  return values[service] || 100;
}
```

### 4.4 Conversion Value Assignment Strategy

#### Value Assignment Framework for Local Services

```javascript
// Comprehensive lead value calculation
const LeadValueCalculator = {

  baseValues: {
    // Service type base values (average job value * close rate)
    'emergency_plumbing': 250,
    'scheduled_plumbing': 150,
    'drain_cleaning': 100,
    'water_heater_repair': 200,
    'water_heater_install': 500,
    'hvac_repair': 300,
    'hvac_maintenance': 100,
    'ac_installation': 2000,
    'furnace_install': 1800,
    'electrical_repair': 200,
    'electrical_install': 400,
    'roof_repair': 800,
    'roof_replacement': 3000
  },

  urgencyMultipliers: {
    'emergency': 1.5,    // Higher close rate, premium pricing
    'same_day': 1.2,     // Good intent
    'this_week': 1.0,    // Standard
    'just_browsing': 0.5 // Lower close rate
  },

  timeOfDayMultipliers: {
    // After hours requests often more valuable
    'business_hours': 1.0,    // 8am - 6pm
    'evening': 1.1,           // 6pm - 10pm
    'late_night': 1.3,        // 10pm - 6am
    'weekend': 1.2
  },

  calculate: function(formData) {
    const service = formData.service || 'scheduled_plumbing';
    const urgency = formData.urgency || 'this_week';

    let value = this.baseValues[service] || 100;
    value *= this.urgencyMultipliers[urgency] || 1.0;
    value *= this.getTimeMultiplier();

    return Math.round(value);
  },

  getTimeMultiplier: function() {
    const hour = new Date().getHours();
    const day = new Date().getDay();

    if (day === 0 || day === 6) return this.timeOfDayMultipliers.weekend;
    if (hour >= 22 || hour < 6) return this.timeOfDayMultipliers.late_night;
    if (hour >= 18) return this.timeOfDayMultipliers.evening;
    return this.timeOfDayMultipliers.business_hours;
  }
};
```

### 4.5 Conversion Window Best Practices

| Conversion Type | Click Window | View Window | Rationale |
|----------------|--------------|-------------|-----------|
| Form Submission | 30 days | 1 day | Local services have longer consideration |
| Phone Call Click | 30 days | 1 day | May call later |
| Completed Phone Call | 30 days | 1 day | Actual lead |
| Form Start (micro) | 7 days | None | Short engagement signal |
| Page Engagement | 7 days | None | Intent signal only |

---

## 5. Phone Call Tracking

### 5.1 Google Forwarding Number Setup

#### Step 1: Enable Call Extensions in Google Ads

1. Ads & Assets > Assets > Call
2. Add Call Extension:
   - Phone number: Your business number
   - Call reporting: ON
   - Count calls as phone call conversions: Checked

#### Step 2: Website Call Tracking Setup

1. Tools & Settings > Measurement > Conversions
2. Create new conversion: Phone Calls > Calls to a phone number on your website
3. Configure:
   - Conversion name: Website Phone Call
   - Call length: 60 seconds (or your threshold)
   - Value: $75
4. Get the tracking snippet

#### Step 3: Implement Website Call Conversion Tracking

Add this script to your landing page (via GTM or direct):

```html
<!-- Google Ads Call Tracking -->
<script>
  (function(a,e,c,f,g,h,b,d){
    var k={ak:"CONVERSION_ID",cl:"CONVERSION_LABEL",autoreplace:"PHONE_NUMBER"};
    a[c]=a[c]||function(){(a[c].q=a[c].q||[]).push(arguments)};
    a[g]||(a[g]=k.ak);
    b=e.createElement(f);
    b.async=1;
    b.src="//www.gstatic.com/wcm/loader.js";
    d=e.getElementsByTagName(f)[0];
    d.parentNode.insertBefore(b,d);
    a[h]=function(b,d,e){a[c](2,b,k,d,null,new Date,e)};
    a["_googWcmGet"]=function(b,d,e){a[c](3,b,k,d,null,new Date,e)}
  })(window,document,"_googWcmImpl","script","_googWcmAk","_googWcmConf498");
</script>
```

Replace:
- `CONVERSION_ID`: Your Google Ads conversion ID (numeric)
- `CONVERSION_LABEL`: Your conversion label
- `PHONE_NUMBER`: Your display phone number format

### 5.2 GTM Implementation for Call Tracking

#### Tag: Google Ads Phone Conversion Snippet

```
Tag Type: Custom HTML
HTML:
<script>
  (function(a,e,c,f,g,h,b,d){
    var k={ak:"XXXXXXXXXX",cl:"YYYYYYYYYYYYYYY",autoreplace:"+1-512-555-1234"};
    a[c]=a[c]||function(){(a[c].q=a[c].q||[]).push(arguments)};
    a[g]||(a[g]=k.ak);
    b=e.createElement(f);
    b.async=1;
    b.src="//www.gstatic.com/wcm/loader.js";
    d=e.getElementsByTagName(f)[0];
    d.parentNode.insertBefore(b,d);
    a[h]=function(b,d,e){a[c](2,b,k,d,null,new Date,e)};
    a["_googWcmGet"]=function(b,d,e){a[c](3,b,k,d,null,new Date,e)}
  })(window,document,"_googWcmImpl","script","_googWcmAk","_googWcmConfXXXXXX");
</script>

Tag firing priority: 100 (fires early)
Triggering: All Pages
```

### 5.3 Click-to-Call Tracking Implementation

#### HTML Structure for Phone Links

```html
<!-- Trackable phone link format -->
<a href="tel:+15125551234"
   class="phone-link"
   data-track="phone-click"
   data-location="header">
  (512) 555-1234
</a>

<!-- With tracking attributes for different locations -->
<a href="tel:+15125551234"
   class="cta-phone hero-phone"
   data-track="phone-click"
   data-location="hero">
  Call Now: (512) 555-1234
</a>

<a href="tel:+15125551234"
   class="sticky-cta-phone"
   data-track="phone-click"
   data-location="sticky_cta">
  Tap to Call
</a>
```

#### JavaScript Click Tracking

```javascript
// Phone click tracking initialization
document.addEventListener('DOMContentLoaded', function() {

  // Track all phone link clicks
  const phoneLinks = document.querySelectorAll('a[href^="tel:"]');

  phoneLinks.forEach(function(link) {
    link.addEventListener('click', function(e) {
      const phoneNumber = this.href.replace('tel:', '');
      const clickLocation = this.dataset.location || 'unknown';

      // Push to data layer
      dataLayer.push({
        'event': 'phone_click',
        'phone_number': phoneNumber,
        'click_location': clickLocation,
        'click_text': this.innerText.trim(),
        'page_url': window.location.href,
        'time_on_page': Math.round((Date.now() - window.pageLoadTime) / 1000)
      });

      // Optional: Add slight delay to ensure tracking fires
      // Only for non-mobile where new window opens
      if (!/Android|iPhone|iPad|iPod/i.test(navigator.userAgent)) {
        e.preventDefault();
        setTimeout(function() {
          window.location.href = link.href;
        }, 150);
      }
    });
  });
});
```

### 5.4 Call Duration Thresholds

#### Recommended Thresholds by Business Type

| Business Type | Minimum Duration | Rationale |
|--------------|------------------|-----------|
| Emergency Services | 30 seconds | Quick dispatch calls |
| Estimates/Quotes | 60 seconds | Need details discussion |
| Complex Services | 120 seconds | Consultation required |
| General Inquiries | 45 seconds | Filter wrong numbers |

#### Configuration in Google Ads

```
For Local Service Businesses:
- Primary Conversion: 60 seconds (qualified lead conversation)
- Secondary Conversion: 30 seconds (any meaningful contact)
- Exclude: < 15 seconds (likely hang-ups/wrong numbers)
```

---

## 6. Form Submission Tracking

### 6.1 Thank You Page Tracking

#### Thank You Page Structure

```html
<!DOCTYPE html>
<html>
<head>
  <title>Thank You - Request Received</title>

  <!-- Data Layer - BEFORE GTM -->
  <script>
    window.dataLayer = window.dataLayer || [];

    // Parse URL parameters for form data
    const urlParams = new URLSearchParams(window.location.search);

    dataLayer.push({
      'event': 'thank_you_page_load',
      'conversion_type': 'form_submission',
      'form_id': urlParams.get('form_id') || 'unknown',
      'form_name': urlParams.get('form_name') || 'Contact Form',
      'service_requested': urlParams.get('service') || 'general',
      'lead_value': parseInt(urlParams.get('value')) || 100,
      'transaction_id': urlParams.get('txn_id') || generateTransactionId(),

      // User data for enhanced conversions
      'user_data': {
        'email': urlParams.get('email') || '',
        'phone_number': urlParams.get('phone') || ''
      }
    });

    function generateTransactionId() {
      return 'lead_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
    }
  </script>

  <!-- GTM Container -->
  <script>
    (function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
    new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
    j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
    'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
    })(window,document,'script','dataLayer','GTM-XXXXXXX');
  </script>
</head>
<body>
  <!-- GTM noscript -->
  <noscript><iframe src="https://www.googletagmanager.com/ns.html?id=GTM-XXXXXXX"
  height="0" width="0" style="display:none;visibility:hidden"></iframe></noscript>

  <main class="thank-you-content">
    <h1>Thank You for Your Request!</h1>
    <p>We've received your information and will contact you shortly.</p>
    <!-- Additional content -->
  </main>
</body>
</html>
```

#### GTM Trigger for Thank You Page

```
Trigger Name: Thank You Page View
Trigger Type: Page View - DOM Ready
This trigger fires on: Some DOM Ready Events
Conditions:
  Page Path contains thank-you
  OR Page Path contains confirmation
  OR Page Path contains success
```

#### GTM Tag for Thank You Page Conversion

```
Tag Type: Google Ads Conversion Tracking
Conversion ID: AW-XXXXXXXXX
Conversion Label: XXXXXXXXXXXXXXXXX
Conversion Value: {{DLV - Lead Value}}
Currency Code: USD
Transaction ID: {{DLV - Transaction ID}}

Triggering: Thank You Page View
```

### 6.2 AJAX Form Tracking (No Page Reload)

#### Complete AJAX Form Implementation

```html
<!-- Form HTML Structure -->
<form id="lead-form"
      class="contact-form"
      data-form-name="Quote Request Form"
      data-form-type="quote">

  <div class="form-group">
    <label for="name">Full Name *</label>
    <input type="text" id="name" name="name" required
           data-field="name" data-track="form-field">
  </div>

  <div class="form-group">
    <label for="email">Email Address *</label>
    <input type="email" id="email" name="email" required
           data-field="email" data-track="form-field">
  </div>

  <div class="form-group">
    <label for="phone">Phone Number *</label>
    <input type="tel" id="phone" name="phone" required
           data-field="phone" data-track="form-field"
           pattern="[\d\s\-\(\)]+">
  </div>

  <div class="form-group">
    <label for="service">Service Needed *</label>
    <select id="service" name="service" required
            data-field="service" data-track="form-field">
      <option value="">Select a service...</option>
      <option value="emergency_plumbing">Emergency Plumbing</option>
      <option value="drain_cleaning">Drain Cleaning</option>
      <option value="water_heater">Water Heater Service</option>
      <option value="general_plumbing">General Plumbing</option>
    </select>
  </div>

  <div class="form-group">
    <label for="urgency">How Soon Do You Need Service?</label>
    <select id="urgency" name="urgency"
            data-field="urgency" data-track="form-field">
      <option value="emergency">Emergency - ASAP</option>
      <option value="same_day">Today</option>
      <option value="this_week">This Week</option>
      <option value="flexible">I'm Flexible</option>
    </select>
  </div>

  <div class="form-group">
    <label for="message">Describe Your Issue</label>
    <textarea id="message" name="message" rows="4"
              data-field="message" data-track="form-field"></textarea>
  </div>

  <div class="form-group">
    <label for="zip">Service ZIP Code *</label>
    <input type="text" id="zip" name="zip" required
           data-field="zip" data-track="form-field"
           pattern="\d{5}">
  </div>

  <button type="submit" class="submit-button">
    Get Free Quote
  </button>
</form>

<div id="form-success" class="hidden">
  <h3>Thank You!</h3>
  <p>We've received your request and will contact you within 30 minutes.</p>
</div>
```

#### JavaScript Form Tracking Handler

```javascript
// ============================================
// AJAX FORM TRACKING IMPLEMENTATION
// ============================================

const FormTracker = {

  // Configuration
  config: {
    formSelector: '#lead-form',
    successElementId: 'form-success',
    submissionEndpoint: '/api/submit-lead',
    enableFieldTracking: true,
    enableFormStartTracking: true
  },

  // State
  state: {
    formStarted: false,
    fieldsCompleted: new Set(),
    startTime: null
  },

  // Initialize
  init: function() {
    const form = document.querySelector(this.config.formSelector);
    if (!form) return;

    this.form = form;
    this.setupFormStartTracking();
    this.setupFieldTracking();
    this.setupSubmitHandler();
  },

  // Track when user first interacts with form
  setupFormStartTracking: function() {
    if (!this.config.enableFormStartTracking) return;

    const self = this;
    const formFields = this.form.querySelectorAll('input, select, textarea');

    formFields.forEach(function(field) {
      field.addEventListener('focus', function() {
        if (!self.state.formStarted) {
          self.state.formStarted = true;
          self.state.startTime = Date.now();

          dataLayer.push({
            'event': 'form_start',
            'form_id': self.form.id,
            'form_name': self.form.dataset.formName,
            'form_type': self.form.dataset.formType,
            'first_field': field.name
          });
        }
      }, { once: false });
    });
  },

  // Track individual field completions
  setupFieldTracking: function() {
    if (!this.config.enableFieldTracking) return;

    const self = this;
    const trackedFields = this.form.querySelectorAll('[data-track="form-field"]');

    trackedFields.forEach(function(field) {
      field.addEventListener('blur', function() {
        if (this.value && !self.state.fieldsCompleted.has(this.name)) {
          self.state.fieldsCompleted.add(this.name);

          dataLayer.push({
            'event': 'form_field_complete',
            'form_id': self.form.id,
            'field_name': this.name,
            'field_type': this.type || this.tagName.toLowerCase(),
            'fields_completed_count': self.state.fieldsCompleted.size
          });
        }
      });
    });
  },

  // Handle form submission
  setupSubmitHandler: function() {
    const self = this;

    this.form.addEventListener('submit', function(e) {
      e.preventDefault();

      if (!self.validateForm()) {
        return;
      }

      const formData = self.collectFormData();
      self.submitForm(formData);
    });
  },

  // Validate form
  validateForm: function() {
    const requiredFields = this.form.querySelectorAll('[required]');
    let isValid = true;

    requiredFields.forEach(function(field) {
      if (!field.value.trim()) {
        field.classList.add('error');
        isValid = false;
      } else {
        field.classList.remove('error');
      }
    });

    return isValid;
  },

  // Collect form data
  collectFormData: function() {
    const formData = new FormData(this.form);
    const data = {};

    formData.forEach(function(value, key) {
      data[key] = value;
    });

    // Add metadata
    data.form_id = this.form.id;
    data.form_name = this.form.dataset.formName;
    data.form_type = this.form.dataset.formType;
    data.time_to_complete = this.state.startTime ?
      Math.round((Date.now() - this.state.startTime) / 1000) : 0;
    data.fields_completed = this.state.fieldsCompleted.size;
    data.transaction_id = this.generateTransactionId();
    data.lead_value = this.calculateLeadValue(data.service, data.urgency);
    data.page_url = window.location.href;
    data.gclid = this.getCookie('gclid') || '';

    return data;
  },

  // Submit form via AJAX
  submitForm: function(formData) {
    const self = this;
    const submitButton = this.form.querySelector('[type="submit"]');

    // Disable button and show loading state
    submitButton.disabled = true;
    submitButton.textContent = 'Submitting...';

    fetch(this.config.submissionEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(formData)
    })
    .then(function(response) {
      if (!response.ok) throw new Error('Submission failed');
      return response.json();
    })
    .then(function(result) {
      self.handleSuccess(formData, result);
    })
    .catch(function(error) {
      self.handleError(error);
      submitButton.disabled = false;
      submitButton.textContent = 'Get Free Quote';
    });
  },

  // Handle successful submission
  handleSuccess: function(formData, result) {
    // Push conversion event to data layer
    dataLayer.push({
      'event': 'form_submission',
      'form_id': formData.form_id,
      'form_name': formData.form_name,
      'form_type': formData.form_type,
      'service_requested': formData.service,
      'urgency': formData.urgency,
      'lead_value': formData.lead_value,
      'transaction_id': formData.transaction_id,
      'time_to_complete': formData.time_to_complete,

      // Enhanced conversions data
      'user_data': {
        'email': formData.email,
        'phone_number': this.normalizePhone(formData.phone),
        'address': {
          'first_name': formData.name ? formData.name.split(' ')[0] : '',
          'last_name': formData.name ? formData.name.split(' ').slice(1).join(' ') : '',
          'postal_code': formData.zip,
          'country': 'US'
        }
      }
    });

    // Show success message
    this.form.classList.add('hidden');
    document.getElementById(this.config.successElementId).classList.remove('hidden');

    // Scroll to success message
    document.getElementById(this.config.successElementId).scrollIntoView({
      behavior: 'smooth'
    });
  },

  // Handle submission error
  handleError: function(error) {
    console.error('Form submission error:', error);

    dataLayer.push({
      'event': 'form_error',
      'form_id': this.form.id,
      'error_message': error.message
    });

    alert('There was an error submitting your request. Please try again or call us directly.');
  },

  // Helper: Calculate lead value
  calculateLeadValue: function(service, urgency) {
    const baseValues = {
      'emergency_plumbing': 250,
      'drain_cleaning': 150,
      'water_heater': 400,
      'general_plumbing': 100
    };

    const urgencyMultipliers = {
      'emergency': 1.5,
      'same_day': 1.2,
      'this_week': 1.0,
      'flexible': 0.8
    };

    const baseValue = baseValues[service] || 100;
    const multiplier = urgencyMultipliers[urgency] || 1.0;

    return Math.round(baseValue * multiplier);
  },

  // Helper: Generate transaction ID
  generateTransactionId: function() {
    return 'lead_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  },

  // Helper: Normalize phone number
  normalizePhone: function(phone) {
    if (!phone) return '';
    const cleaned = phone.replace(/\D/g, '');
    return cleaned.length === 10 ? '+1' + cleaned : '+' + cleaned;
  },

  // Helper: Get cookie value
  getCookie: function(name) {
    const match = document.cookie.match(new RegExp('(^| )' + name + '=([^;]+)'));
    return match ? match[2] : null;
  }
};

// Initialize on DOM ready
document.addEventListener('DOMContentLoaded', function() {
  FormTracker.init();
});
```

### 6.3 Lead Quality Scoring

#### Client-Side Lead Scoring

```javascript
// ============================================
// LEAD QUALITY SCORING SYSTEM
// ============================================

const LeadScorer = {

  // Scoring weights
  weights: {
    // Service urgency
    urgency: {
      'emergency': 30,
      'same_day': 25,
      'this_week': 15,
      'flexible': 5
    },

    // Service type value
    serviceValue: {
      'emergency_plumbing': 25,
      'water_heater': 25,
      'ac_installation': 30,
      'hvac_repair': 20,
      'drain_cleaning': 15,
      'general_plumbing': 10
    },

    // Form completeness
    fieldsProvided: {
      'name': 5,
      'email': 10,
      'phone': 15,
      'address': 10,
      'message': 10,
      'zip': 5
    },

    // Engagement signals
    engagement: {
      'scroll_depth_90': 10,
      'time_on_page_120s': 10,
      'multiple_pages': 5,
      'return_visitor': 10
    }
  },

  // Calculate lead score
  calculate: function(formData, engagementData) {
    let score = 0;
    const scoreBreakdown = {};

    // Score urgency
    if (formData.urgency && this.weights.urgency[formData.urgency]) {
      score += this.weights.urgency[formData.urgency];
      scoreBreakdown.urgency = this.weights.urgency[formData.urgency];
    }

    // Score service type
    if (formData.service && this.weights.serviceValue[formData.service]) {
      score += this.weights.serviceValue[formData.service];
      scoreBreakdown.service = this.weights.serviceValue[formData.service];
    }

    // Score form completeness
    scoreBreakdown.fields = 0;
    Object.keys(this.weights.fieldsProvided).forEach(field => {
      if (formData[field] && formData[field].trim()) {
        score += this.weights.fieldsProvided[field];
        scoreBreakdown.fields += this.weights.fieldsProvided[field];
      }
    });

    // Score engagement
    scoreBreakdown.engagement = 0;
    if (engagementData) {
      if (engagementData.scrollDepth >= 90) {
        score += this.weights.engagement.scroll_depth_90;
        scoreBreakdown.engagement += this.weights.engagement.scroll_depth_90;
      }
      if (engagementData.timeOnPage >= 120) {
        score += this.weights.engagement.time_on_page_120s;
        scoreBreakdown.engagement += this.weights.engagement.time_on_page_120s;
      }
      if (engagementData.isReturnVisitor) {
        score += this.weights.engagement.return_visitor;
        scoreBreakdown.engagement += this.weights.engagement.return_visitor;
      }
    }

    return {
      score: score,
      maxScore: 150,
      percentage: Math.round((score / 150) * 100),
      grade: this.getGrade(score),
      breakdown: scoreBreakdown
    };
  },

  // Get letter grade
  getGrade: function(score) {
    if (score >= 100) return 'A';
    if (score >= 80) return 'B';
    if (score >= 60) return 'C';
    if (score >= 40) return 'D';
    return 'F';
  },

  // Get lead tier for routing
  getTier: function(score) {
    if (score >= 100) return 'hot';      // Immediate callback priority
    if (score >= 70) return 'warm';      // Same-day callback
    if (score >= 40) return 'standard';  // Normal queue
    return 'nurture';                     // Email sequence
  }
};

// Integration with form submission
function onFormSubmit(formData) {
  const engagementData = {
    scrollDepth: TrackingEvents.getCurrentScrollDepth(),
    timeOnPage: TrackingEvents.getTimeOnPage(),
    isReturnVisitor: document.cookie.includes('returning_visitor=true')
  };

  const leadScore = LeadScorer.calculate(formData, engagementData);

  // Add score to form data
  formData.lead_score = leadScore.score;
  formData.lead_grade = leadScore.grade;
  formData.lead_tier = LeadScorer.getTier(leadScore.score);

  // Push to data layer
  dataLayer.push({
    'event': 'lead_scored',
    'lead_score': leadScore.score,
    'lead_grade': leadScore.grade,
    'lead_tier': LeadScorer.getTier(leadScore.score),
    'score_breakdown': leadScore.breakdown
  });

  return formData;
}
```

---

## 7. Remarketing Setup

### 7.1 Google Ads Remarketing Tag

#### GTM Tag Configuration

```
Tag Name: Google Ads - Remarketing Tag
Tag Type: Google Ads Remarketing

Google Ads Conversion ID: AW-XXXXXXXXX

Custom Parameters:
  ecomm_prodid: {{DLV - Service Category}}
  ecomm_pagetype: {{DLV - Page Type}}
  ecomm_totalvalue: {{DLV - Lead Value}}

Triggering: All Pages
```

#### Dynamic Remarketing Parameters

```javascript
// Data layer for remarketing parameters
dataLayer.push({
  'event': 'remarketing_data',
  'google_tag_params': {
    'ecomm_prodid': 'plumbing_emergency',        // Service category
    'ecomm_pagetype': 'product',                  // landing = product view
    'ecomm_totalvalue': 250,                      // Potential value
    'service_type': 'emergency',
    'service_area': 'austin_central'
  }
});
```

### 7.2 Audience Creation in Google Ads

#### Step 1: Access Audience Manager

1. Tools & Settings > Shared Library > Audience Manager
2. Click "+" to create new audience segment

#### Audience Segment 1: All Landing Page Visitors

```
Segment Name: LP Visitors - All
Segment Type: Website visitors
Segment Members: Visitors of a page

Rules:
  URL contains /landing
  OR URL contains /services
  OR URL matches regex ^https://.*\/(plumbing|hvac|electrical).*$

Membership Duration: 30 days
Initial List Size: Include visitors from past 30 days
```

#### Audience Segment 2: Form Starters (Intent Signals)

```
Segment Name: LP - Form Starters
Segment Type: Website visitors
Segment Members: Visitors of a page with specific tag

Rules:
  Event equals form_start

Membership Duration: 14 days
Initial List Size: Include past visitors
```

#### Audience Segment 3: Engaged Non-Converters

```
Segment Name: LP - Engaged Non-Converters
Segment Type: Website visitors
Segment Members: Custom combination

Include:
  Visitors where scroll_depth >= 75
  OR Visitors where time_on_site >= 60 seconds

Exclude:
  LP - Converters (form submissions)

Membership Duration: 21 days
```

#### Audience Segment 4: Form Abandoners

```
Segment Name: LP - Form Abandoners
Segment Type: Website visitors
Segment Members: Custom combination

Include:
  Visitors where event equals form_start

Exclude:
  Visitors where event equals form_submission

Membership Duration: 7 days
```

#### Audience Segment 5: Converters (Exclusion List)

```
Segment Name: LP - Converters (Exclude)
Segment Type: Website visitors
Segment Members: Visitors of a page with specific tag

Rules:
  Event equals form_submission
  OR Page URL contains thank-you
  OR Page URL contains confirmation

Membership Duration: 90 days
```

#### Audience Segment 6: High-Value Service Interest

```
Segment Name: LP - High Value Interest
Segment Type: Website visitors
Segment Members: Custom combination

Rules:
  service_category equals emergency_plumbing
  OR service_category equals water_heater
  OR service_category equals hvac_installation

Membership Duration: 30 days
```

### 7.3 GTM Audience Trigger Tags

#### Tag: Remarketing - Form Start Audience

```
Tag Type: Custom HTML
HTML:
<script>
  gtag('event', 'form_starter', {
    'send_to': 'AW-XXXXXXXXX',
    'dynx_itemid': '{{DLV - Service Category}}',
    'dynx_pagetype': 'form_start'
  });
</script>

Triggering: Form Start
```

#### Tag: Remarketing - High Engagement Audience

```
Tag Type: Custom HTML
HTML:
<script>
  gtag('event', 'engaged_visitor', {
    'send_to': 'AW-XXXXXXXXX',
    'dynx_itemid': '{{DLV - Service Category}}',
    'dynx_pagetype': 'engaged',
    'scroll_depth': '{{Scroll Depth Threshold}}'
  });
</script>

Triggering: Scroll Depth - 75%
```

### 7.4 Customer Match Integration

#### Uploading Offline Conversions for Similar Audiences

```javascript
// Server-side: Format customer data for upload
const formatCustomerData = (customer) => {
  return {
    email: customer.email.toLowerCase().trim(),
    phone: normalizePhone(customer.phone),
    firstName: customer.firstName.toLowerCase().trim(),
    lastName: customer.lastName.toLowerCase().trim(),
    zipCode: customer.zip,
    country: 'US'
  };
};

// Customer Match audiences to create:
// 1. Past customers (for exclusion and lookalike)
// 2. High-value customers (for lookalike targeting)
// 3. Service-specific customers (for cross-sell)
```

---

## 8. Quality Score Optimization

### 8.1 Landing Page Experience Requirements

#### Technical Requirements Checklist

| Requirement | Target | Measurement |
|------------|--------|-------------|
| **Mobile-Friendly** | 100% responsive | Google Mobile-Friendly Test |
| **Page Speed (Mobile)** | LCP < 2.5s | PageSpeed Insights |
| **Page Speed (Desktop)** | LCP < 1.5s | PageSpeed Insights |
| **HTTPS** | Required | SSL Certificate |
| **No Intrusive Interstitials** | Pass | Manual review |
| **Crawlable Content** | All text visible | Fetch as Google |

#### Content Relevance Checklist

| Element | Requirement | Implementation |
|---------|-------------|----------------|
| **Headline** | Matches ad copy and keywords | Include primary keyword |
| **Subheadline** | Supports main offer | Value proposition |
| **Body Copy** | Relevant to search intent | Address user needs |
| **CTA** | Clear next step | Action-oriented text |
| **Trust Signals** | Credibility indicators | Reviews, certifications |
| **Contact Options** | Multiple channels | Phone, form, chat |

### 8.2 Page Speed Implementation

#### Critical Performance Optimizations

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Emergency Plumbing Services | Same-Day Service</title>

  <!-- Preconnect to critical third-party origins -->
  <link rel="preconnect" href="https://www.googletagmanager.com">
  <link rel="preconnect" href="https://www.google-analytics.com">
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

  <!-- Preload critical assets -->
  <link rel="preload" href="/css/critical.css" as="style">
  <link rel="preload" href="/fonts/main-font.woff2" as="font" type="font/woff2" crossorigin>
  <link rel="preload" href="/images/hero-image.webp" as="image">

  <!-- Critical CSS inline -->
  <style>
    /* Include above-the-fold critical CSS here */
    /* Approximately 10-15KB maximum */

    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: system-ui, -apple-system, sans-serif;
      line-height: 1.6;
    }

    .hero {
      min-height: 500px;
      display: flex;
      align-items: center;
      background: #1a365d;
      color: white;
    }

    .container {
      max-width: 1200px;
      margin: 0 auto;
      padding: 0 20px;
    }

    h1 {
      font-size: clamp(1.75rem, 4vw, 3rem);
      margin-bottom: 1rem;
    }

    .cta-button {
      display: inline-block;
      padding: 16px 32px;
      background: #f6ad55;
      color: #1a365d;
      text-decoration: none;
      font-weight: bold;
      border-radius: 4px;
    }

    /* Critical form styles */
    .lead-form {
      background: white;
      padding: 24px;
      border-radius: 8px;
    }

    .form-group {
      margin-bottom: 16px;
    }

    .form-group label {
      display: block;
      margin-bottom: 4px;
      font-weight: 500;
    }

    .form-group input,
    .form-group select,
    .form-group textarea {
      width: 100%;
      padding: 12px;
      border: 1px solid #ccc;
      border-radius: 4px;
      font-size: 16px;
    }
  </style>

  <!-- Non-critical CSS loaded asynchronously -->
  <link rel="stylesheet" href="/css/main.css" media="print" onload="this.media='all'">
  <noscript><link rel="stylesheet" href="/css/main.css"></noscript>

  <!-- Data Layer initialization (minimal) -->
  <script>
    window.dataLayer = window.dataLayer || [];
  </script>
</head>
<body>

  <!-- Above-the-fold content loads immediately -->
  <header><!-- Navigation --></header>

  <section class="hero">
    <div class="container">
      <h1>24/7 Emergency Plumbing Services</h1>
      <p>Fast response. Fair prices. Licensed professionals.</p>
      <a href="tel:+15125551234" class="cta-button">Call Now: (512) 555-1234</a>
    </div>
  </section>

  <!-- Main content -->
  <main>
    <!-- Form section -->
    <!-- Service details -->
    <!-- Trust signals -->
  </main>

  <footer><!-- Footer content --></footer>

  <!-- Defer non-critical JavaScript -->
  <script src="/js/main.js" defer></script>

  <!-- GTM loaded after critical content -->
  <script>
    // Load GTM after page becomes interactive
    window.addEventListener('load', function() {
      setTimeout(function() {
        (function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
        new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
        j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
        'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
        })(window,document,'script','dataLayer','GTM-XXXXXXX');
      }, 100);
    });
  </script>

</body>
</html>
```

#### Image Optimization

```html
<!-- Responsive images with modern formats -->
<picture>
  <!-- WebP for modern browsers -->
  <source
    type="image/webp"
    srcset="/images/hero-400.webp 400w,
            /images/hero-800.webp 800w,
            /images/hero-1200.webp 1200w"
    sizes="(max-width: 600px) 100vw, 50vw">

  <!-- JPEG fallback -->
  <img
    src="/images/hero-800.jpg"
    srcset="/images/hero-400.jpg 400w,
            /images/hero-800.jpg 800w,
            /images/hero-1200.jpg 1200w"
    sizes="(max-width: 600px) 100vw, 50vw"
    alt="Licensed plumber fixing emergency leak"
    width="800"
    height="600"
    loading="lazy"
    decoding="async">
</picture>

<!-- Hero image should not be lazy loaded -->
<img
  src="/images/hero.webp"
  alt="24/7 Emergency Plumbing"
  width="1200"
  height="600"
  fetchpriority="high">
```

### 8.3 Core Web Vitals Targets

| Metric | Good | Needs Improvement | Poor |
|--------|------|-------------------|------|
| **LCP** (Largest Contentful Paint) | ≤ 2.5s | 2.5s - 4.0s | > 4.0s |
| **INP** (Interaction to Next Paint) | ≤ 200ms | 200ms - 500ms | > 500ms |
| **CLS** (Cumulative Layout Shift) | ≤ 0.1 | 0.1 - 0.25 | > 0.25 |

#### Monitoring Core Web Vitals

```javascript
// Core Web Vitals tracking with web-vitals library
import { onLCP, onINP, onCLS } from 'web-vitals';

function sendToAnalytics(metric) {
  dataLayer.push({
    'event': 'core_web_vital',
    'metric_name': metric.name,
    'metric_value': metric.value,
    'metric_rating': metric.rating, // 'good', 'needs-improvement', 'poor'
    'metric_delta': metric.delta,
    'metric_id': metric.id
  });
}

onLCP(sendToAnalytics);
onINP(sendToAnalytics);
onCLS(sendToAnalytics);
```

### 8.4 Mobile-Friendliness Requirements

#### Mobile-Specific Implementation

```css
/* Mobile-first responsive design */

/* Touch targets minimum 48x48px */
.cta-button,
.form-submit,
a[href^="tel:"] {
  min-height: 48px;
  min-width: 48px;
  padding: 14px 24px;
}

/* Readable font sizes */
body {
  font-size: 16px; /* Minimum for mobile */
  line-height: 1.6;
}

/* Form inputs sized for mobile */
input, select, textarea {
  font-size: 16px; /* Prevents zoom on iOS */
  padding: 14px;
}

/* Adequate spacing between clickable elements */
.nav-link,
.form-group {
  margin-bottom: 16px;
}

/* Responsive container */
.container {
  width: 100%;
  max-width: 1200px;
  padding: 0 16px;
  margin: 0 auto;
}

/* Mobile-specific adjustments */
@media (max-width: 768px) {
  h1 {
    font-size: 1.75rem;
  }

  .hero {
    padding: 40px 0;
    text-align: center;
  }

  .phone-cta {
    display: block;
    width: 100%;
    text-align: center;
    position: sticky;
    bottom: 0;
    z-index: 100;
    padding: 16px;
    background: #f6ad55;
  }
}
```

---

## 9. Testing and Validation

### 9.1 GTM Preview Mode Testing

#### Test Checklist

```
PRE-LAUNCH TESTING CHECKLIST

[ ] GTM Container
    [ ] Container loads on all pages
    [ ] No JavaScript errors in console
    [ ] Preview mode shows all expected tags

[ ] Data Layer
    [ ] page_data_ready event fires on load
    [ ] All variables populate correctly
    [ ] Campaign parameters captured from URL

[ ] GA4 Tracking
    [ ] Config tag fires on all pages
    [ ] Page views recorded correctly
    [ ] Custom events fire on triggers
    [ ] User properties set correctly

[ ] Google Ads Conversions
    [ ] Conversion Linker fires on all pages
    [ ] Form submission conversion fires
    [ ] Phone click conversion fires
    [ ] Conversion values pass correctly
    [ ] Enhanced conversions data included

[ ] Phone Tracking
    [ ] Click-to-call events fire
    [ ] Phone numbers tracked correctly
    [ ] Google forwarding numbers display

[ ] Form Tracking
    [ ] form_start fires on first interaction
    [ ] form_submission fires on submit
    [ ] All form fields captured
    [ ] AJAX forms track without page reload
    [ ] Thank you page tracking works

[ ] Remarketing
    [ ] Remarketing tag fires on all pages
    [ ] Custom parameters included
    [ ] Audience triggers fire correctly

[ ] Cross-Browser Testing
    [ ] Chrome (desktop + mobile)
    [ ] Safari (desktop + iOS)
    [ ] Firefox
    [ ] Edge
    [ ] Samsung Internet
```

### 9.2 Google Tag Assistant Verification

#### Steps to Verify

1. Install Google Tag Assistant Legacy Chrome extension
2. Navigate to landing page
3. Click extension icon
4. Review tag status:
   - Green: Working correctly
   - Blue: Minor issues
   - Yellow: Warnings
   - Red: Errors

#### Common Issues and Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| Tag not firing | Trigger misconfiguration | Check trigger conditions in preview mode |
| Missing conversion value | Variable not populated | Verify data layer variable path |
| Duplicate tags | Multiple containers | Remove duplicate GTM installations |
| Slow tag loading | Blocking scripts | Move GTM to async loading |

### 9.3 Real-Time Reports Validation

#### GA4 Real-Time Testing

1. Open GA4 > Reports > Realtime
2. Navigate to landing page in separate window
3. Verify in Realtime:
   - User count increases
   - Page title appears
   - Events fire (scroll, form_start, etc.)
   - Conversions register

#### Google Ads Conversion Testing

1. Navigate to landing page with GCLID parameter:
   `?gclid=test_click_id_12345`
2. Complete test conversion
3. Check Tools & Settings > Conversions
4. Verify "Recording conversions" status

### 9.4 Debug Mode Implementation

```javascript
// Debug mode for development/testing
const DebugMode = {

  enabled: false,

  init: function() {
    // Enable via URL parameter or cookie
    this.enabled =
      new URLSearchParams(window.location.search).has('debug') ||
      document.cookie.includes('tracking_debug=true');

    if (this.enabled) {
      this.enableDebug();
    }
  },

  enableDebug: function() {
    console.log('%c[Tracking Debug Mode Enabled]', 'color: green; font-weight: bold;');

    // Log all data layer pushes
    const originalPush = dataLayer.push;
    dataLayer.push = function() {
      console.log('%c[dataLayer.push]', 'color: blue;', arguments[0]);
      return originalPush.apply(dataLayer, arguments);
    };

    // Add visual indicator
    this.addDebugIndicator();
  },

  addDebugIndicator: function() {
    const indicator = document.createElement('div');
    indicator.innerHTML = 'DEBUG MODE';
    indicator.style.cssText = `
      position: fixed;
      bottom: 10px;
      right: 10px;
      background: red;
      color: white;
      padding: 8px 16px;
      font-size: 12px;
      font-weight: bold;
      z-index: 99999;
      border-radius: 4px;
    `;
    document.body.appendChild(indicator);
  },

  log: function(category, message, data) {
    if (!this.enabled) return;
    console.log(
      `%c[${category}]%c ${message}`,
      'color: purple; font-weight: bold;',
      'color: inherit;',
      data || ''
    );
  }
};

// Initialize on page load
document.addEventListener('DOMContentLoaded', function() {
  DebugMode.init();
});
```

---

## 10. Troubleshooting Guide

### 10.1 Common Issues and Solutions

#### Issue: Conversions Not Recording

**Symptoms:**
- Google Ads shows no conversions
- GA4 shows form submissions but Ads doesn't

**Diagnostic Steps:**
1. Check GTM Preview Mode - is conversion tag firing?
2. Verify Conversion Linker is firing on all pages
3. Check for GCLID in cookies
4. Verify conversion ID and label match Google Ads

**Solutions:**
```javascript
// Verify GCLID storage
console.log('GCLID Cookie:', document.cookie.match(/gclid=([^;]+)/));
console.log('GCLID URL:', new URLSearchParams(location.search).get('gclid'));

// Check conversion linker
console.log('_gcl cookies:', document.cookie.match(/_gcl[a-z]*=([^;]+)/g));
```

#### Issue: Duplicate Conversions

**Symptoms:**
- Conversion count higher than actual submissions
- Same conversion recorded multiple times

**Solutions:**
```javascript
// Add transaction ID deduplication
const transactionId = 'lead_' + Date.now() + '_' + formId;

// Store in session to prevent duplicates
if (sessionStorage.getItem('converted_' + formId)) {
  console.log('Duplicate conversion prevented');
  return;
}
sessionStorage.setItem('converted_' + formId, transactionId);

// Include transaction ID in conversion
dataLayer.push({
  'event': 'form_submission',
  'transaction_id': transactionId
  // ... other data
});
```

#### Issue: Enhanced Conversions Not Working

**Symptoms:**
- Enhanced conversions show "Not recording" in Google Ads
- User data not being captured

**Diagnostic Steps:**
1. Verify enhanced conversions enabled in Google Ads conversion settings
2. Check GTM tag has "Include user-provided data" enabled
3. Verify data layer contains user_data object
4. Check data format matches Google requirements

**Solutions:**
```javascript
// Verify user data format
const userData = {
  'email': 'test@example.com',           // Must be lowercase
  'phone_number': '+15125551234',         // Must include country code
  'address': {
    'first_name': 'john',                 // Must be lowercase
    'last_name': 'doe',                   // Must be lowercase
    'postal_code': '78701',
    'country': 'US'                       // 2-letter code
  }
};

console.log('User data for enhanced conversions:', userData);
```

#### Issue: Phone Call Tracking Not Working

**Symptoms:**
- Google forwarding numbers not appearing
- Call conversions not recording

**Solutions:**
1. Verify call conversion snippet is loading
2. Check phone number format matches exactly
3. Ensure page is receiving Google Ads traffic

```javascript
// Debug phone tracking
console.log('WCM loaded:', typeof _googWcmGet !== 'undefined');

// Force refresh of forwarding number
if (typeof _googWcmGet !== 'undefined') {
  _googWcmGet(function(formattedNumber, unformattedNumber) {
    console.log('Forwarding number:', formattedNumber);
  }, '+1-512-555-1234');
}
```

### 10.2 Data Quality Monitoring

#### Weekly Audit Checklist

```
WEEKLY TRACKING AUDIT

Date: ___________
Auditor: ___________

DATA COLLECTION
[ ] GA4 receiving data (check Realtime)
[ ] Event counts within normal range
[ ] No sampling issues in reports
[ ] All custom events firing

CONVERSIONS
[ ] Form submission count matches CRM leads
[ ] Phone click count reasonable
[ ] Conversion values accurate
[ ] Attribution data complete

QUALITY INDICATORS
[ ] Bounce rate within normal range (< 70%)
[ ] Average session duration normal (> 30s)
[ ] No unusual traffic spikes
[ ] Geographic distribution normal

TAG HEALTH
[ ] GTM container version current
[ ] No error tags in preview mode
[ ] All triggers functioning
[ ] No deprecated tags

NOTES:
_________________________________
_________________________________
_________________________________
```

#### Automated Monitoring Setup

```javascript
// Automated data quality checks
const DataQualityMonitor = {

  checks: {
    // Check for unusual bounce rate
    bounceRate: function(rate) {
      if (rate > 0.85) {
        this.alert('High bounce rate detected: ' + (rate * 100) + '%');
      }
    },

    // Check for conversion tracking
    conversionRate: function(sessions, conversions) {
      const rate = conversions / sessions;
      if (rate < 0.001) { // Less than 0.1%
        this.alert('Conversion rate unusually low - check tracking');
      }
      if (rate > 0.5) { // More than 50%
        this.alert('Conversion rate unusually high - possible duplicate tracking');
      }
    },

    // Check for data freshness
    dataFreshness: function(lastEventTime) {
      const hoursSinceLastEvent = (Date.now() - lastEventTime) / (1000 * 60 * 60);
      if (hoursSinceLastEvent > 24) {
        this.alert('No events recorded in 24+ hours');
      }
    }
  },

  alert: function(message) {
    // Send alert via your preferred method
    console.error('[Data Quality Alert]', message);

    // Optional: Push to data layer for logging
    dataLayer.push({
      'event': 'data_quality_alert',
      'alert_message': message,
      'alert_timestamp': new Date().toISOString()
    });
  }
};
```

### 10.3 Tag Firing Sequence

#### Optimal Tag Loading Order

```
1. Conversion Linker (priority: 100)
   └── Fires first to capture GCLID

2. GA4 Configuration (priority: 90)
   └── Establishes session tracking

3. Google Ads Remarketing (priority: 80)
   └── Captures visitor for audiences

4. Event-based tags (priority: default)
   └── Fire based on user interactions

5. Conversion tags (priority: default)
   └── Fire on conversion events
```

#### Implementing Tag Priority in GTM

```
Tag Priority Settings:
- Higher number = fires earlier
- Default priority = 0
- Set Conversion Linker to 100
- Set Config tags to 90
- Leave event tags at default
```

---

## Appendix A: Complete GTM Container Export

### Import Instructions

1. Download the JSON configuration below
2. In GTM, go to Admin > Import Container
3. Choose "Merge" and select "Rename conflicting tags"
4. Review and publish

### Container Configuration Summary

```
Tags (12):
1. GA4 - Configuration
2. GA4 - Form Submission Event
3. GA4 - Phone Click Event
4. GA4 - Scroll Depth Event
5. GA4 - Form Start Event
6. Google Ads - Conversion Linker
7. Google Ads - Form Submission Conversion
8. Google Ads - Phone Click Conversion
9. Google Ads - Remarketing Tag
10. Google Ads - Phone Call Tracking
11. Custom HTML - Debug Mode
12. Custom HTML - Data Layer Validation

Triggers (11):
1. All Pages
2. Form Submission (Custom Event)
3. Phone Click (Custom Event)
4. Click to Call (Link Click)
5. Scroll Depth 25%
6. Scroll Depth 50%
7. Scroll Depth 75%
8. Scroll Depth 90%
9. Thank You Page
10. Form Start (Custom Event)
11. DOM Ready

Variables (15):
1-10. Data Layer Variables (as specified above)
11. Service Category Value (Lookup Table)
12. GCLID from Cookie
13. Page Type
14. Transaction ID
15. Debug Mode Enabled
```

---

## Appendix B: Implementation Timeline

### Recommended Implementation Schedule

| Week | Tasks | Owner | Deliverables |
|------|-------|-------|--------------|
| **Week 1** | GTM Setup, Data Layer | Developer | Container installed, data layer live |
| **Week 2** | GA4 Configuration | Analytics | Property configured, events tracking |
| **Week 3** | Google Ads Conversions | Marketing | Conversion actions live |
| **Week 4** | Phone & Form Tracking | Developer | All conversions tracking |
| **Week 5** | Remarketing Setup | Marketing | Audiences created |
| **Week 6** | Testing & Validation | QA | Full audit complete |
| **Week 7** | Optimization | All | Performance benchmarks met |

---

## Appendix C: Measurement Plan Template

### Business Objectives to KPIs Mapping

| Business Objective | KPI | Target | Measurement Method |
|-------------------|-----|--------|-------------------|
| Generate qualified leads | Form submissions | 50/month | GA4 + Google Ads conversion |
| Drive phone inquiries | Phone clicks + calls | 100/month | Call tracking + click events |
| Maximize ad ROI | Cost per lead | < $75 | Google Ads + CRM integration |
| Improve lead quality | Lead score average | > 70 | Custom lead scoring system |
| Increase engagement | Scroll depth 75%+ | > 40% | GA4 scroll tracking |

---

## Document Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | January 2026 | Analytics Team | Initial release |

---

**Implementation Support**

For implementation questions or troubleshooting assistance:
- Google Tag Manager Help: https://support.google.com/tagmanager
- GA4 Documentation: https://support.google.com/analytics
- Google Ads Help: https://support.google.com/google-ads

**End of Document**

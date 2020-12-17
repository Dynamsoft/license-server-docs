---
layout: default-layout
title: How a license is tracked
keywords: license tracking, mechanism
description: This page describes how license tracking is done
breadcrumbText: Mechanism
needAutoGenerateSidebar: true
---

# How a license is tracked

All Dynamsoft SDKs that support trackable licenses have a built-in mechanism to get authorization at initialization as well as collect and report its usage during runtime.

## Configure LTS

> Read more on [What is LTS]({{site.about}}terms.html#license-tracking-server)

If you are using Dynamsoft-hosting LTS, then you can skip this step as the SDK has been configured to connect to the LTS hosted by Dynamsoft by default. Please read [Configure the Handshake code](#configure-the-handshake-code).

If you have hosted LTS on your own server, you can configure the connection like this

* JavaScript

``` javascript
Dynamsoft.DBR.BarcodeReader.licenseServer = ["https://your.mainServer.com", "https://your.backupServer.com"];
```

* C++

``` cpp
DM_LTSConnectionParameters ltspar;    
reader.InitLTSConnectionParameters(&ltspar);
ltspar.mainServerURL = "https://lts.yoursite.com";
```

* CSharp

``` csharp
DMLTSConnectionParameters ltspar = _br.InitLTSConnectionParamters();           
ltspar.MainServerURL = "https://lts.yoursite.com";
```

* Java

``` java
BarcodeReader br = new BarcodeReader("")
DMLTSConnectionParameters ltspar = br.initLTSConnectionParameters();
ltspar.mainServerURL = "https://lts.yoursite.com";
```

* Python

``` python
BarcodeReader br = new BarcodeReader();
DMLTSConnectionParameters ltspar = reader.init_lts_connection_parameters();
ltspar.mainServerURL = "https://lts.yoursite.com";
```

On Android

``` java
DBRLTSLicenseVerificationListener ltsListener = new DBRLTSLicenseVerificationListener() {
    @Override
    public void LTSLicenseVerificationCallback(boolean success, Exception error) {
        Assert.assertEquals(false, success);
        Assert.assertEquals("ChargeWay for licenseItem is not matched.", error.getMessage());
    }
};
DMLTSConnectionParameters parameters = new DMLTSConnectionParameters();
parameters.mainServerURL = "https://lts.yoursite.com";
```

On iOS

``` c
iDMLTSConnectionParameters* lts = [[iDMLTSConnectionParameters alloc] init];
lts.mainServerURL = @"https://lts.yoursite.com";
```

## Configure the Handshake code

* JavaScript

``` javascript
Dynamsoft.DBR.BarcodeReader.handshakeCode = "DynamsoftID-CustomCode";
let reader = await Dynamsoft.DBR.BarcodeReader.createInstance();
```

* C++

``` cpp
DM_LTSConnectionParameters ltspar;    
reader.InitLTSConnectionParameters(&ltspar);
ltspar.handshakeCode = "Your-HandshakeCode";
iRet = reader.InitLicenseFromLTS(&ltspar,szErrorMsg,256);
```

* CSharp

``` csharp
DMLTSConnectionParameters ltspar = _br.InitLTSConnectionParamters();           
ltspar.HandshakeCode = "Your-HandshakeCode";
EnumErrorCode iRet = BarcodeReader.InitLicenseFromLTS(ltspar, out strErrorMSG);
```

* Java

``` java
BarcodeReader br = new BarcodeReader("")
DMLTSConnectionParameters ltspar = br.initLTSConnectionParameters();
ltspar.handshakeCode = "Your-HandshakeCode";
br.initLicenseFromLTS(ltspar);
```

* Python

``` python
BarcodeReader reader = new BarcodeReader();
DMLTSConnectionParameters ltspar = reader.init_lts_connection_parameters();
ltspar.handshakeCode = "Your-HandshakeCode";
iRet = reader.init_license_from_lts(ltspar);
```


On Android

``` java
DBRLTSLicenseVerificationListener ltsListener = new DBRLTSLicenseVerificationListener() {
    @Override
    public void LTSLicenseVerificationCallback(boolean success, Exception error) {
        Assert.assertEquals(false, success);
        Assert.assertEquals("ChargeWay for licenseItem is not matched.", error.getMessage());
    }
};
DMLTSConnectionParameters parameters = new DMLTSConnectionParameters();
parameters.handshakeCode = "Your-HandshakeCode";
dbr.initLicenseFromLTS(parameters,ltsListener);
```

On iOS

``` c
iDMLTSConnectionParameters* lts = [[iDMLTSConnectionParameters alloc] init];
lts.handshakeCode = @"Your-HandshakeCode";
_dbr = [[DynamsoftBarcodeReader alloc] initLicenseFromLTS:lts verificationDelegate:self];

* (void)LTSLicenseVerificationCallback:(bool)isSuccess error:(NSError * _Nullable)error

{
    NSNumber* boolNumber = [NSNumber numberWithBool:isSuccess];
    dispatch_async(dispatch_get_main_queue(), ^{
        [self->m_verificationReceiver performSelector:self->m_verificationCallback withObject:boolNumber withObject:error];
        NSLog(@"ifsuccess : %@",boolNumber);
        NSLog(@"error code: %ld:",(long)error.code);
        NSLog(@"errormsg : %@",error.userInfo);
    });
}
```

## Authorization

During initialization, Dynamsoft SDKs connect to `LTS` to get authorized. The process is simplified as follows

* The SDK sends a request to `LTS`
* `LTS` extracts the handshake code from the request and checks its status in the usage database
* If the handshake code exists and its quota is not used up,  `LTS` returns a token
* Otherwise, a failure message is returned

### About the request

The request includes the Client UUID and the handshake code. The handshake code is fetched from your code, the UUID is fetched from cache or generated on the fly if it is not found in the cache.

### Handle authorization error

For better user experience, you need to handle the authorization error should it happens. Take the JavaScript edition for example

``` javascript
try {
    reader = await Dynamsoft.DBR.BarcodeReader.createInstance();
} catch (ex) {
    // Check the error message and if it's a license error, like
    // "License has exceeded its limit."
    // Try to notify the user to try again or to contact the site administrator
}
```

## License Tracking

### Dynamsoft Barcode Reader

The main purpose of the Barcode Reader is to locate and decode barcodes, therefore, its usage is about successful barcode scans.

Each successful scan is recorded and a minimum report is generated every 3 minutes, the report is submitted every 3 minutes too. We'll dive deeper into the mechanism below.

#### Usage report

The report includes the following fields

* Time label (indicates which 3-minute slot it covers)
* handshake code
* Client UUID
* Count of barcodes
* Type of barcodes

### About time

* 3-minute for reports

Dynamsoft uses absolute time when generating usage reports. This means, each time slot is fixed, for example, the first time slot is always 00:00:01 ~ 00:03:00 and the second is 00:03:01 ~ 00:06:00 and so on. Each barcode scan has a time stamp of its own and is counted in the 3-minute window in which it is done.

* 3-minute for submitting reports

Unlike the time used for reporting, report submitting is better done in a more timely manner. Therefore, the moment Dynamsoft SDKs initializes, it'll check whether there are any usage reports left unsubmitted and submit all of them at once, after that, it'll submit the next report after 3 minutes, and so on and so forth.

### Report submission

Dynamsoft SDKs submit usage reports to LTS with HTTP Post requests. If the submission succeeds, it'll remove the local copy of the report, otherwise, the report remains on the device and will be submitted again the next time the SDK initializes. Note that the SDK won't attempt to submit failed reports during the same session.

### Report analysis

LTS receives usage reports from client devices and does the math to make overall usage reports. These reports are done per each handshake code. Read more on [statistics page]({{site.about}}statistics-page.html).

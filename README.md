# UBL 2.1 DIAN

Core for electronic invoicing pre-validation - DIAN UBL 2.1.

# Tags
* 1.0: Contains valid tests with binary security token (SOAP) and XAdES signature (XML) with algorithms sha1, sha256 and sha512.
* 1.1: Contains main templates for Web Service consumption, require curl as a dependency.
* 1.1.1: Canonization error is solved.
* 1.2: Contains valid proofs for the sending of credit notes and calculation of the CUDE.
* 1.3: License LGPL.
* 2.0: Contains valid proofs for the sending of debit notes and document name standard.

# Resources

## Documentation

Última actualización Nov 16, 2019 at 3:09PM | Publicado en Jun 30, 2019

Núcleo de Facturación Electrónica Validación Previa DIAN UBL 2.1.

### Tags:

-   1.0: Contiene pruebas válidas con el token de seguridad binario
    (SOAP) y la firma XAdES (XML) con los algoritmos sha1, sha256 y
    sha512.
-   1.1: Contiene las plantillas principales para el consumo del
    servicio web, requiere curl como una dependencia.
-   11.1.1: Se soluciona el error de canonización.
-   1.2: Contiene pruebas válidas para el envío de notas de crédito y el
    cálculo del CUDE.
-   1.3: Licencia LGPL.
-   2.0: Contiene pruebas válidas para el envío de notas de débito y el
    nombre del documento estándar.

### Cómo instalar:

Instalar con [composer](https://getcomposer.org/)

``` {.wp-block-code}
composer require 3creativessas/tc-ubl21dian
```

### SOAP uso básico:

``` {.wp-block-code}
use DOMDocument;
use Stenfrank\UBL21dian\BinarySecurityToken\SOAP;

$xmlString = <<<XML
<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope" xmlns:wcf="http://wcf.dian.colombia">
    <soap:Header/>
    <soap:Body>
        <wcf:GetStatus>
            <!--Optional:-->
            <wcf:trackId>123456666</wcf:trackId>
        </wcf:GetStatus>
    </soap:Body>
</soap:Envelope>
XML;

$pathCertificate = 'PATH_CERTIFICATE.p12';
$passwors = 'PASSWORS_CERTIFICATE';

$domDocument = new DOMDocument();
$domDocument->loadXML($xmlString);

$soap = new SOAP($pathCertificate, $passwors);
$soap->Action = 'http://wcf.dian.colombia/IWcfDianCustomerServices/GetStatus';

$soap->sign($domDocument->saveXML());

file_put_contents('./SOAPDIAN21.xml', $soap->xml);
```

### XAdES SHA1 uso básico:

``` {.wp-block-code}
use DOMDocument;
use Stenfrank\UBL21dian\XAdES\SignInvoice;

$xmlString = <<<XML
<?xml version="1.0" encoding="UTF-8"?>
<Invoice>
    <ext:UBLExtensions>
        <ext:UBLExtension>
            <ext:ExtensionContent/>
        </ext:UBLExtension>
        <ext:UBLExtension>
            <ext:ExtensionContent/>
        </ext:UBLExtension>
    </ext:UBLExtensions>
</Invoice>
XML;

$pathCertificate = 'PATH_CERTIFICATE.p12';
$passwors = 'PASSWORS_CERTIFICATE';

$domDocument = new DOMDocument();
$domDocument->loadXML($xmlString);

$signInvoice = new SignInvoice($pathCertificate, $passwors, $xmlString, SignInvoice::ALGO_SHA1);

file_put_contents('./SING1.xml', $signInvoice->xml);
```

### XAdES SHA256 uso básico:

``` {.wp-block-code}
use DOMDocument;
use Stenfrank\UBL21dian\XAdES\SignInvoice;

$xmlString = <<<XML
<?xml version="1.0" encoding="UTF-8"?>
<Invoice>
    <ext:UBLExtensions>
        <ext:UBLExtension>
            <ext:ExtensionContent/>
        </ext:UBLExtension>
        <ext:UBLExtension>
            <ext:ExtensionContent/>
        </ext:UBLExtension>
    </ext:UBLExtensions>
</Invoice>
XML;

$pathCertificate = 'PATH_CERTIFICATE.p12';
$passwors = 'PASSWORS_CERTIFICATE';

$domDocument = new DOMDocument();
$domDocument->loadXML($xmlString);

$signInvoice = new SignInvoice($pathCertificate, $passwors, $xmlString);

file_put_contents('./SING256.xml', $signInvoice->xml);
```

### XAdES SHA512 uso básico:

``` {.wp-block-code}
use DOMDocument;
use Stenfrank\UBL21dian\XAdES\SignInvoice;

$xmlString = <<<XML
<?xml version="1.0" encoding="UTF-8"?>
<Invoice>
    <ext:UBLExtensions>
        <ext:UBLExtension>
            <ext:ExtensionContent/>
        </ext:UBLExtension>
        <ext:UBLExtension>
            <ext:ExtensionContent/>
        </ext:UBLExtension>
    </ext:UBLExtensions>
</Invoice>
XML;

$pathCertificate = 'PATH_CERTIFICATE.p12';
$passwors = 'PASSWORS_CERTIFICATE';

$domDocument = new DOMDocument();
$domDocument->loadXML($xmlString);

$signInvoice = new SignInvoice($pathCertificate, $passwors, $xmlString, SignInvoice::ALGO_SHA512);

file_put_contents('./SING512.xml', $signInvoice->xml);
```

### Calcular el código de seguridad del software:

``` {.wp-block-code}
// If you assign the values "softwareID" and "pin" the library will calculate and assign "Software Security Code" at the moment of signing the document.
$xadesDIAN->softwareID = 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx';
$xadesDIAN->pin = '12345';
```

### Calcular «UUID» (CUFE):

``` {.wp-block-code}
// If you assign the value "technicalKey" the library will calculate and assign "UUID" (CUFE) at the moment of signing the document
$xadesDIAN->technicalKey = 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx';
```

### Calcular «UUID» (CUDE):

``` {.wp-block-code}
// If you assign the value "pin" the library will calculate and assign "UUID" (CUDE) at the moment of signing the document
$signCreditNote->pin = 'xxxxx';
```

### Consumo de servicios web (Cliente): {#web-service}

``` {.wp-block-code}
use Stenfrank\UBL21dian\Client;
use Stenfrank\UBL21dian\Templates\SOAP\GetStatusZip;

$pathCertificate = 'PATH_CERTIFICATE.p12';
$passwors = 'PASSWORS_CERTIFICATE';

$getStatusZip = new GetStatusZip($pathCertificate, $passwors);
$getStatusZip->trackId = 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx';

// Sign
$getStatusZip->sign();

$client = new Client($getStatusZip);

// DIAN Response Web Service
return $client;
```

### Consumo de servicios web (Plantilla):

``` {.wp-block-code}
use Stenfrank\UBL21dian\Templates\SOAP\GetStatusZip;

$pathCertificate = 'PATH_CERTIFICATE.p12';
$passwors = 'PASSWORS_CERTIFICATE';

$getStatusZip = new GetStatusZip($pathCertificate, $passwors);
$getStatusZip->trackId = 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx';

// Sign to send - DIAN Response Web Service
return $getStatusZip->signToSend();
```


## Authors

* **Frank Aguirre** - *Maintainer* - [Stenfrank](https://github.com/Stenfrank/)
* **Helbert Arias** - *Accomplice* - [helbertDev3c](https://github.com/helbertDev3c/)

## Donation
If this library help you reduce time to develop, you can give me a cup of coffee :smiley:.

[![paypal](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.me/stenfrank/1?locale.x=es_XC)

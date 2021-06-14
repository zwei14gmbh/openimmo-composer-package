---
description: >-
  Generated classes for reading and writing, (de)serializing and validating 
  OpenImmo XML files
---

# Readme

The package contains classes \(generated with `goetas-webservices/xsd2php`\) for reading and writing, \(de\)serializing \(with `jms/serializer`\) and validating OpenImmo files. 

### Installation

```text
composer require zwei14/openimmo:1.0.x-dev
```

### Regenerate OpenImmo classes

Regenerate OpenImmo classes after installation.

```text
# Step 1
./../../../vendor/bin/xsd2php convert openimmo.yml /path/to/openimmo_127b.xsd

# Step 2
git apply patch_user_defined_simplefield.patch

# Step 3
composer dumpautoload
```

{% hint style="danger" %}
Patch fixes `user_defined_simplefield` class and corresponding `*.yml` file for JMS serializer. Otherwise values are missing in XML after reading or writing \(e.g. in import\).
{% endhint %}

### Regenerate OpenImmo-Feedback classes

```text
# Step 1
./../../../vendor/bin/xsd2php convert openimmo_feedback.yml /path/to/openimmo-feedback_125.xsd

# Step 2
composer dumpautoload
```

### Examples

#### Reading OpenImmo XML \(plain\)

```php
use GoetasWebservices\Xsd\XsdToPhpRuntime\Jms\Handler\BaseTypesHandler;
use GoetasWebservices\Xsd\XsdToPhpRuntime\Jms\Handler\XmlSchemaDateHandler;
use JMS\Serializer\Handler\HandlerRegistryInterface;
use JMS\Serializer\SerializerBuilder as JmsSerializerBuilder;
use JMS\Serializer\Serializer as JmsSerializer;

[...]

$xmlString = file_get_contents('/path/to/foobar.xml');

$jmsSerializerBuilder = JmsSerializerBuilder::create();
$jmsSerializerBuilder->addMetadataDir(
    '/path/to/vendor/zwei14/openimmo/metadata/Zwei14/OpenImmo/API',     
    'Zwei14\OpenImmo\API'
);

$jmsSerializerBuilder->configureHandlers(function (HandlerRegistryInterface $handler) use ($jmsSerializerBuilder) {
    $jmsSerializerBuilder->addDefaultHandlers();
    $handler->registerSubscribingHandler(new BaseTypesHandler()); // XMLSchema List handling
    $handler->registerSubscribingHandler(new XmlSchemaDateHandler()); // XMLSchema date handling
    // $handler->registerSubscribingHandler(new YourhandlerHere());
});

/* @var $jmsSerializer JmsSerializer */
$jmsSerializer = $jmsSerializerBuilder->build();

/* @var $openImmo Openimmo */
$openImmo = $jmsSerializer->deserialize($xmlString, Openimmo::class, 'xml');
```

#### Reading OpenImmo XML using SerializerTrait

```php
use SerializerTrait;

/* @var $jmsSerializer JmsSerializer */
$jmsSerializer = $this->getOpenImmoJmsSerializer('/path/to/vendor');

/* @var $openImmo Openimmo */
$openImmo = $jmsSerializer->deserialize($xmlString, Openimmo::class, 'xml');
```

#### Writing OpenImmo XML

```php
$infrastruktur = new Infrastruktur();

$ausblick = new Ausblick();
$ausblick->setBlick('BERGE');

$distanzenSport = [
    new DistanzenSport(15.0),
    new DistanzenSport(10.0),
];
$distanzenSport[0]->setDistanzZuSport('SEE');
$distanzenSport[1]->setDistanzZuSport('SPORTANLAGEN');

$distanzen = [
    new Distanzen(1.0),
];
$distanzen[0]->setDistanzZu('AUTOBAHN');

$infrastruktur
    ->setZulieferung(false)
    ->setAusblick($ausblick)
    ->setDistanzenSport($distanzenSport)
    ->setDistanzen($distanzen);
    
/* @var $jmsSerializer JmsSerializer */
$jmsSerializer = $this->getOpenImmoJmsSerializer('/path/to/vendor');

$newXml = $jmsSerializer->serialize($infrastruktur, 'xml');
```

**Result**

```markup
<infrastruktur>
    <zulieferung>false</zulieferung>
        <ausblick blick="BERGE"/>
        <distanzen distanz_zu="AUTOBAHN">1.0</distanzen>
        <distanzen_sport distanz_zu_sport="SEE">15.0</distanzen_sport>
        <distanzen_sport distanz_zu_sport="SPORTANLAGEN">10.0</distanzen_sport>
</infrastruktur>
```


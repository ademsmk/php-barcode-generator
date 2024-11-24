# PHP Barkod Oluşturucu

<a href="https://packagist.org/packages/picqer/php-barkod-oluşturucu"><img src="https://img.shields.io/packagist/dt/picqer/php-barkod-oluşturucu" alt="Toplam İndirmeler"></a>
<a href="https://packagist.org/packages/picqer/php-barkod-oluşturucu"><img src="https://img.shields.io/packagist/v/picqer/php-barkod-oluşturucu" alt="En Son Kararlı Sürüm"></a>

Bu, PHP'de kullanımı kolay, şişkin olmayan, çerçeveden bağımsız bir barkod oluşturucudur. Sıfır(!) besteci bağımlılığı kullanır ve yalnızca bir avuç dosyadan oluşur. Muhtemelen bu, Packagist'te PHP için en çok indirilen barkod oluşturucu olmasının nedenidir. ;)

En çok kullanılan 1D barkod standartlarından SVG, PNG, JPG ve HTML resimleri oluşturur.

## ... için destek yok
- QR kodları gibi herhangi bir **2D** barkod için destek yok.
- Sadece barkodun 'çubuklar' kısmını, barkodun altında metin olmadan oluşturuyoruz. Barkodun altında kodun metnini istiyorsanız, bunu daha sonra bu paketin çıktısına ekleyebilirsiniz.

## Kurulum
[composer](https://getcomposer.org/doc/00-intro.md) aracılığıyla kurulum yapın:

```
composer require picqer/php-barcode-generator
```

PNG veya JPG görüntüleri oluşturmak istiyorsanız, sisteminizde GD kütüphanesi veya Imagick'in de yüklü olması gerekir. SVG veya HTML render'ları için hiçbir bağımlılık yoktur.

## Kullanım
Belirli bir görüntü biçiminde (örneğin PNG veya SVG) belirli bir "tür" (örneğin Kod 128 veya UPC) için bir barkod istersiniz.

- İlk olarak, barkodunu istediğiniz dizeyi barkod türlerinden biriyle bir `Barkod` nesnesine kodlayın.
- Ardından, `Barkod` nesnesindeki çubukların görüntüsünü oluşturmak için render'lardan birini kullanın.

> "Tür", hangi karakterleri kodlayabileceğinizi ve hangi çubukların hangi karakteri temsil ettiğini tanımlayan bir standarttır. En çok kullanılan türler [kod 128](https://en.wikipedia.org/wiki/Code_128) ve [EAN/UPC](https://en.wikipedia.org/wiki/International_Article_Number)'dır. Her karakter her barkod türüne kodlanamaz ve tüm barkod tarayıcıları tüm türleri okuyamaz.

```php
<?php
require 'vendor/autoload.php';

// Code128 kodlamasının Barkod nesnesini yap.
$barcode = (new Picqer\Barcode\Types\TypeCode128())->getBarcode('081231723897');

// Barkodu HTML Oluşturucu ile tarayıcıda HTML olarak çıktı olarak al
$renderer = new Picqer\Barcode\Renderers\HtmlRenderer();
echo $renderer->render($barcode);
```

Bu güzellikle sonuçlanacaktır:<br>
![Barkod 081231723897 Kod 128 olarak](tests/verified-files/081231723897-ean13.svg)

Her bir görüntü oluşturucunun kendi seçenekleri vardır. Örneğin, bir PNG'nin yüksekliğini, genişliğini ve rengini ayarlayabilirsiniz:
```php
<?php
require 'vendor/autoload.php';

$colorRed = [255, 0, 0];

$barcode = (new Picqer\Barcode\Types\TypeCode128())->getBarcode('081231723897');
$renderer = new Picqer\Barcode\Renderers\PngRenderer();
$renderer->setForegroundColor($colorRed);

// PNG'yi dosya sistemine, widthFactor 3 (barkodun genişliği x 3) ve yüksekliği 50 piksel olacak şekilde kaydedin
file_put_contents('barcode.png', $renderer->render($barcode, $barcode->getWidth() * 3, 50));
```

## Görüntü işleyiciler
Mevcut görüntü işleyiciler: SVG, PNG, JPG ve HTML.

Hepsi RendererInterface'e uygundur ve aynı `render()` yöntemine sahiptir. Bazı işleyicilerin set*() yöntemleri aracılığıyla ek seçenekleri de vardır.

### Genişlikler
render() yöntemi Barkod nesnesine, genişliğe ve yüksekliğe ihtiyaç duyar. **JPG/PNG görüntüleri için**, yalnızca Barkod nesnesinin genişliğinin bir faktörü olan bir genişlik verdiğinizde geçerli bir barkod elde edersiniz. İşte bu yüzden örnekler, görüntüyü barkod verilerinin genişliğinden 2 kat daha geniş yapmak için `$barcode->getWidth() * 2` gösterir. Genişlik olarak rastgele bir sayı *verebilirsiniz* ve görüntü mümkün olan en iyi şekilde ölçeklenir, ancak kenar yumuşatma olmadan, mükemmel bir şekilde geçerli olmaz.

HTML ve SVG işleyicileri herhangi bir genişliği ve yüksekliği, hatta kayan noktaları bile işleyebilir.

Her işleyici için tüm seçenekler şunlardır:

### PNG + JPG
All options for PNG and JPG are the same.
```php
$renderer = new Picqer\Barcode\Renderers\PngRenderer();
$renderer->setForegroundColor([255, 0, 0]); // Give a color for the bars, default is black. Give it as 3 times 0-255 values for red, green and blue. 
$renderer->setBackgroundColor([0, 255, 255]); // Give a color for the background, default is transparent (in PNG) or white (in JPG). Give it as 3 times 0-255 values for red, green and blue. 
$renderer->useGd(); // If you have Imagick and GD installed, but want to use GD
$renderer->useImagick(); // If you have Imagick and GD installed, but want to use Imagick

$renderer->render($barcode, 5, 40); // Width factor (how many pixel wide every bar is), and the height in pixels
````

### HTML
Gives HTML to use inline in a full HTML document.
```php
$renderer = new Picqer\Barcode\Renderers\HtmlRenderer();
$renderer->setForegroundColor([255, 0, 0]); // Give a color red for the bars, default is black. Give it as 3 times 0-255 values for red, green and blue. 
$renderer->setBackgroundColor([0, 0, 255]); // Give a color blue for the background, default is transparent. Give it as 3 times 0-255 values for red, green and blue. 

$renderer->render($barcode, 450.20, 75); // Width and height support floats
````

### Dynamic HTML
Give HTML here the barcode is using the full width and height, to put inside a container/div that has a fixed size.
```php
$renderer = new Picqer\Barcode\Renderers\DynamicHtmlRenderer();
$renderer->setForegroundColor([255, 0, 0]); // Give a color red for the bars, default is black. Give it as 3 times 0-255 values for red, green and blue. 
$renderer->setBackgroundColor([0, 0, 255]); // Give a color blue for the background, default is transparent. Give it as 3 times 0-255 values for red, green and blue. 

$renderer->render($barcode);
````

You can put the rendered HTML inside a div like this:
```html
<div style="width: 400px; height: 75px"><?php echo $renderedBarcode; ?></div>
```

## Accepted barcode types
These barcode types are supported. All types support different character sets and some have mandatory lengths. Please see wikipedia for supported chars and lengths per type.

You can find all supported types in the [src/Types](src/Types) folder.

Most used types are TYPE_CODE_128 and TYPE_CODE_39. Because of the best scanner support, variable length and most chars supported.

- TYPE_CODE_32 (italian pharmaceutical code 'MINSAN')
- TYPE_CODE_39
- TYPE_CODE_39_CHECKSUM
- TYPE_CODE_39E
- TYPE_CODE_39E_CHECKSUM
- TYPE_CODE_93
- TYPE_STANDARD_2_5
- TYPE_STANDARD_2_5_CHECKSUM
- TYPE_INTERLEAVED_2_5
- TYPE_INTERLEAVED_2_5_CHECKSUM
- TYPE_CODE_128
- TYPE_CODE_128_A
- TYPE_CODE_128_B
- TYPE_CODE_128_C
- TYPE_EAN_2
- TYPE_EAN_5
- TYPE_EAN_8
- TYPE_EAN_13
- TYPE_ITF14 (Also known as GTIN-14)
- TYPE_UPC_A
- TYPE_UPC_E
- TYPE_MSI
- TYPE_MSI_CHECKSUM
- TYPE_POSTNET
- TYPE_PLANET
- TYPE_RMS4CC
- TYPE_KIX
- TYPE_IMB
- TYPE_CODABAR
- TYPE_CODE_11
- TYPE_PHARMA_CODE
- TYPE_PHARMA_CODE_TWO_TRACKS

[See example images for all supported barcode types](examples.md)

## A note about PNG and JPG images
If you want to use PNG or JPG images, you need to install [Imagick](https://www.php.net/manual/en/intro.imagick.php) or the [GD library](https://www.php.net/manual/en/intro.image.php). This package will use Imagick if that is installed, or fall back to GD. If you have both installed, but you want a specific method, you can use `$renderer->useGd()` or `$renderer->useImagick()` to force your preference.

## Examples

### Embedded PNG image in HTML
```php
$barcode = (new Picqer\Barcode\Types\TypeCode128())->getBarcode('081231723897');
$renderer = new Picqer\Barcode\Renderers\PngRenderer();
echo '<img src="data:image/png;base64,' . base64_encode($renderer->render($barcode, $barcode->getWidth() * 2)) . '">';
```

### Save JPG barcode to disk
```php
$barcode = (new Picqer\Barcode\Types\TypeCodabar())->getBarcode('081231723897');
$renderer = new Picqer\Barcode\Renderers\JpgRenderer();

file_put_contents('barcode.jpg', $renderer->render($barcode, $barcode->getWidth() * 2));
```

### Oneliner SVG output to disk
```php
file_put_contents('barcode.svg', (new Picqer\Barcode\Renderers\SvgRenderer())->render((new Picqer\Barcode\Types\TypeKix())->getBarcode('6825ME601')));
```

## Upgrading to v3
There is no need to change anything when upgrading from v2 to v3. Above you find the new preferred way of using this library since v3. But the old style still works.

To give the renderers the same interface, setting colors is now always with an array of RGB colors. If you use the old BarcodeGenerator* classes and use colors with names ('red') or hex codes (#3399ef), these will be converted using the ColorHelper. All hexcodes are supported, but for names of colors only the basic colors are supported.

If you want to convert to the new style, here is an example:
```php
// Old style
$generator = new Picqer\Barcode\BarcodeGeneratorSVG();
echo $generator->getBarcode('081231723897', $generator::TYPE_CODE_128);

// New style
$barcode = (new Picqer\Barcode\Types\TypeCode128())->getBarcode('081231723897');
$renderer = new Picqer\Barcode\Renderers\SvgRenderer();
echo $renderer->render($barcode);
```

The width in the renderer is now the width of the end result, instead of the widthFactor. If you want to keep dynamic widths, you can get the width of the encoded Barcode and multiply it by the widthFactor to get the same result as before. See here an example for a widthFactor of 2:
```php
// Old style
$generator = new Picqer\Barcode\BarcodeGeneratorSVG();
echo $generator->getBarcode('081231723897', $generator::TYPE_CODE_128, 2. 30);

// New style
$barcode = (new Picqer\Barcode\Types\TypeCode128())->getBarcode('081231723897');
$renderer = new Picqer\Barcode\Renderers\SvgRenderer();
echo $renderer->render($barcode, $barcode->getWidth() * 2, 30);
```

---

## Previous style generators
In version 3 the barcode type encoders and image renderers are completely separate. This makes building your own renderer way easier. The old way was using "generators". Below are the old examples of these generators, which still works in v3 as well.

### Usage
Initiate the barcode generator for the output you want, then call the ->getBarcode() routine as many times as you want.

```php
<?php
require 'vendor/autoload.php';

// This will output the barcode as HTML output to display in the browser
$generator = new Picqer\Barcode\BarcodeGeneratorHTML();
echo $generator->getBarcode('081231723897', $generator::TYPE_CODE_128);
```

Will result in this beauty:<br>
![Barcode 081231723897 as Code 128](tests/verified-files/081231723897-ean13.svg)

The `getBarcode()` method accepts the following parameters:
- `$barcode` String needed to encode in the barcode
- `$type` Type of barcode, use the constants defined in the class
- `$widthFactor` Width is based on the length of the data, with this factor you can make the barcode bars wider than default
- `$height` The total height of the barcode in pixels
- `$foregroundColor` Hex code as string, or array of RGB, of the colors of the bars (the foreground color)

Example of usage of all parameters:

```php
<?php

require 'vendor/autoload.php';

$redColor = [255, 0, 0];

$generator = new Picqer\Barcode\BarcodeGeneratorPNG();
file_put_contents('barcode.png', $generator->getBarcode('081231723897', $generator::TYPE_CODE_128, 3, 50, $redColor));
```

### Image types
```php
$generatorSVG = new Picqer\Barcode\BarcodeGeneratorSVG(); // Vector based SVG
$generatorPNG = new Picqer\Barcode\BarcodeGeneratorPNG(); // Pixel based PNG
$generatorJPG = new Picqer\Barcode\BarcodeGeneratorJPG(); // Pixel based JPG
$generatorHTML = new Picqer\Barcode\BarcodeGeneratorHTML(); // Pixel based HTML
$generatorHTML = new Picqer\Barcode\BarcodeGeneratorDynamicHTML(); // Vector based HTML
```

#### Embedded PNG image in HTML
```php
$generator = new Picqer\Barcode\BarcodeGeneratorPNG();
echo '<img src="data:image/png;base64,' . base64_encode($generator->getBarcode('081231723897', $generator::TYPE_CODE_128)) . '">';
```

#### Save JPG barcode to disk
```php
$generator = new Picqer\Barcode\BarcodeGeneratorJPG();
file_put_contents('barcode.jpg', $generator->getBarcode('081231723897', $generator::TYPE_CODABAR));
```

#### Oneliner SVG output to disk
```php
file_put_contents('barcode.svg', (new Picqer\Barcode\BarcodeGeneratorSVG())->getBarcode('6825ME601', Picqer\Barcode\BarcodeGeneratorSVG::TYPE_KIX));
```

---
````

### PNG + JPG
PNG ve JPG için tüm seçenekler aynıdır. ```php
$renderer = new Picqer\Barcode\Renderers\PngRenderer();
$renderer->setForegroundColor([255, 0, 0]); // Çubuklar için bir renk verin, varsayılan bl'dir

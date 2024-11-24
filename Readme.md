# PHP Barkod Oluşturucu
<a href="https://github.com/picqer/php-barkod-oluşturucu/actions"><img src="https://github.com/picqer/php-barkod-oluşturucu/workflows/phpunit/badge.svg" alt="Derleme Durumu"></a>
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

### SVG
Vektör tabanlı bir SVG resmi. Yazdırmak için en iyi kaliteyi verir.
```php
$renderer = new Picqer\Barcode\Renderers\SvgRenderer();
$renderer->setForegroundColor([255, 0, 0]); // Çubuklar için kırmızı bir renk verin, varsayılan siyahtır. Kırmızı, yeşil ve mavi için 3 kez 0-255 değerleri verin.
$renderer->setBackgroundColor([0, 0, 255]); // Arkaplan için mavi bir renk verin, varsayılan şeffaftır. Kırmızı, yeşil ve mavi için 3 kez 0-255 değerleri verin.
$renderer->setSvgType($renderer::TYPE_SVG_INLINE); // Çıktının, tek başına bir SVG resmi yerine HTML belgelerinin içinde satır içi olarak kullanılmasını değiştirir (varsayılan)
$renderer->setSvgType($renderer::TYPE_SVG_STANDALONE); // Varsayılanı zorlamak istiyorsanız, tek başına bir SVG resmi oluşturun

$renderer->render($barcode, 450.20, 75); // Genişlik ve yükseklik, kayan noktaları destekler
````

### PNG + JPG
PNG ve JPG için tüm seçenekler aynıdır. ```php
$renderer = new Picqer\Barcode\Renderers\PngRenderer();
$renderer->setForegroundColor([255, 0, 0]); // Çubuklar için bir renk verin, varsayılan bl'dir

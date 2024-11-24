# PHP Barcode Generator 

<a href="https://packagist.org/packages/picqer/php-barcode-generator"><img src="https://img.shields.io/packagist/dt/picqer/php-barcode-generator" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/picqer/php-barcode-generator"><img src="https://img.shields.io/packagist/v/picqer/php-barcode-generator" alt="Latest Stable Version"></a>

Bu, PHP'de kullanımı kolay, şişkin olmayan, çerçeveden bağımsız bir barkod oluşturucudur. Sıfır(!) besteci bağımlılığı kullanır ve yalnızca bir avuç dosyadan oluşur. Muhtemelen bu, Packagist'te PHP için en çok indirilen barkod oluşturucu olmasının nedenidir. ;)

En çok kullanılan 1D barkod standartlarından SVG, PNG, JPG ve HTML resimleri oluşturur.

## ... için destek yok
- QR kodları gibi herhangi bir **2D** barkod için destek yok.
- Sadece barkodun 'çubuklar' kısmını, barkodun altında metin olmadan oluşturuyoruz. Barkodun altında kodun metnini istiyorsanız, bunu daha sonra bu paketin çıktısına ekleyebilirsiniz.

[composer](https://getcomposer.org/doc/00-intro.md) aracılığıyla yükleyin:

```
composer require picqer/php-barcode-generator
```

PNG veya JPG görüntüleri oluşturmak istiyorsanız, sisteminizde GD kütüphanesi veya Imagick'in de yüklü olması gerekir. SVG veya HTML render'ları için herhangi bir bağımlılık yoktur.

Kullanım
belirli bir "tür" (örneğin Kod 128 veya UPC) için belirli bir görüntü bölümü (örneğin PNG veya SVG) bir barkod yönetimi.

İlk olarak, barkodunu istediğiniz dizeyi barkod türlerinden yazılımların Barkodnesnelerine kodlayın.
Ayrıca, Barkodnesnedeki çubukların düzenlemek için işleyicilerden birini kullanın.
"Tür", hangi karakterleri kodlayabileceğinizi ve hangi çubukların hangi karakteri temsil ettiğini kanıtlayabileceğiniz bir standarttır. En çok kullanılan türler kod 128 ve EAN/UPC'dir . Her karakteri, her barkod tipi kodlanamaz ve tüm barkod tarayıcıları, tüm türleri okuyamaz.

<?php
require 'vendor/autoload.php';

// Make Barcode object of Code128 encoding.
$barcode = (new Picqer\Barcode\Types\TypeCode128())->getBarcode('081231723897');

// Output the barcode as HTML in the browser with a HTML Renderer
$renderer = new Picqer\Barcode\Renderers\HtmlRenderer();
echo $renderer->render($barcode);
Bu güzellikle sonuçlanacak:
Barkod 081231723897, Kod 128 olarak

Her bir renderer'ın kendine özgü seçenekleri vardır. Örneğin, bir PNG'nin yüksekliğini, genişliğini ve rengini ayarlayabilirsiniz:

<?php
require 'vendor/autoload.php';

$colorRed = [255, 0, 0];

$barcode = (new Picqer\Barcode\Types\TypeCode128())->getBarcode('081231723897');
$renderer = new Picqer\Barcode\Renderers\PngRenderer();
$renderer->setForegroundColor($colorRed);

// Save PNG to the filesystem, with widthFactor 3 (width of the barcode x 3) and height of 50 pixels
file_put_contents('barcode.png', $renderer->render($barcode, $barcode->getWidth() * 3, 50));
Görüntü oluşturucular
Mevcut resim oluşturucular: SVG, PNG, JPG ve HTML.

Hepsi RendererInterface'e uygundur ve aynı render()metoda sahiptir. Bazı renderer'ların set*() metodları aracılığıyla ek seçenekleri de vardır.

Genişlikler
render() metodu Barkod nesnesine, genişliğe ve yüksekliğe ihtiyaç duyar. JPG/PNG görüntüleri için , yalnızca Barkod nesnesinin genişliğinin bir faktörü olan bir genişlik verdiğinizde geçerli bir barkod elde edersiniz. Bu nedenle örnekler, $barcode->getWidth() * 2görüntüyü barkod verilerinin genişliğinden 2 kat daha geniş pikseller halinde yapmayı gösterir. Genişlik olarak keyfi bir sayı verebilirsiniz ve görüntü mümkün olan en iyi şekilde ölçeklenir, ancak kenar yumuşatma olmadan mükemmel bir şekilde geçerli olmaz.

HTML ve SVG görüntüleyicileri her genişlik ve yüksekliği, hatta kayan yazıları bile işleyebilir.

Her bir render aracı için tüm seçenekler şunlardır:

SVG
Vektör tabanlı bir SVG resmi. Baskıda en iyi kaliteyi verir.

$renderer = new Picqer\Barcode\Renderers\SvgRenderer();
$renderer->setForegroundColor([255, 0, 0]); // Give a color red for the bars, default is black. Give it as 3 times 0-255 values for red, green and blue. 
$renderer->setBackgroundColor([0, 0, 255]); // Give a color blue for the background, default is transparent. Give it as 3 times 0-255 values for red, green and blue. 
$renderer->setSvgType($renderer::TYPE_SVG_INLINE); // Changes the output to be used inline inside HTML documents, instead of a standalone SVG image (default)
$renderer->setSvgType($renderer::TYPE_SVG_STANDALONE); // If you want to force the default, create a stand alone SVG image

$renderer->render($barcode, 450.20, 75); // Width and height support floats
PNG + JPG
PNG ve JPG için tüm seçenekler aynıdır.

$renderer = new Picqer\Barcode\Renderers\PngRenderer();
$renderer->setForegroundColor([255, 0, 0]); // Give a color for the bars, default is black. Give it as 3 times 0-255 values for red, green and blue. 
$renderer->setBackgroundColor([0, 255, 255]); // Give a color for the background, default is transparent (in PNG) or white (in JPG). Give it as 3 times 0-255 values for red, green and blue. 
$renderer->useGd(); // If you have Imagick and GD installed, but want to use GD
$renderer->useImagick(); // If you have Imagick and GD installed, but want to use Imagick

$renderer->render($barcode, 5, 40); // Width factor (how many pixel wide every bar is), and the height in pixels
HTML
Tam bir HTML belgesinde satır içi kullanılacak HTML kodunu verir.

$renderer = new Picqer\Barcode\Renderers\HtmlRenderer();
$renderer->setForegroundColor([255, 0, 0]); // Give a color red for the bars, default is black. Give it as 3 times 0-255 values for red, green and blue. 
$renderer->setBackgroundColor([0, 0, 255]); // Give a color blue for the background, default is transparent. Give it as 3 times 0-255 values for red, green and blue. 

$renderer->render($barcode, 450.20, 75); // Width and height support floats
Dinamik HTML
Burada barkodun tam genişliğini ve yüksekliğini kullanarak sabit boyutlu bir container/div içerisine yerleştirilmesi için HTML kodunu verin.

$renderer = new Picqer\Barcode\Renderers\DynamicHtmlRenderer();
$renderer->setForegroundColor([255, 0, 0]); // Give a color red for the bars, default is black. Give it as 3 times 0-255 values for red, green and blue. 
$renderer->setBackgroundColor([0, 0, 255]); // Give a color blue for the background, default is transparent. Give it as 3 times 0-255 values for red, green and blue. 

$renderer->render($barcode);
Oluşturulan HTML'i şu şekilde bir div elemanının içine koyabilirsiniz:

<div style="width: 400px; height: 75px"><?php echo $renderedBarcode; ?></div>
Kabul edilen barkod türleri
Bu barkod türleri desteklenir. Tüm türler farklı karakter setlerini destekler ve bazılarının zorunlu uzunlukları vardır. Lütfen desteklenen karakterler ve tür başına uzunluklar için Wikipedia'ya bakın.

Desteklenen tüm türleri src/Types klasöründe bulabilirsiniz .

En çok kullanılan tipler TYPE_CODE_128 ve TYPE_CODE_39'dur. En iyi tarayıcı desteği, değişken uzunluk ve desteklenen en fazla karakter nedeniyle.

TYPE_CODE_32 (İtalyan ilaç kodu 'MINSAN')
TÜR_KODU_39
TÜR_KODU_39_KONTROL_TOPLAMI
TÜR_KODU_39E
TÜR_KODU_39E_KONTROL_TOPLAMI
TÜR_KODU_93
TÜR_STANDART_2_5
TÜR_STANDART_2_5_KONTROL_TOPLAMI
TÜR_ARALIKLI_2_5
TÜR_ARALIKLI_2_5_KONTROL_TOPLAMI
TÜR_KODU_128
TÜR_KODU_128_A
TÜR_KODU_128_B
TÜR_KODU_128_C
TÜR_EAN_2
TÜR_EAN_5
TÜR_EAN_8
TÜR_EAN_13
TYPE_ITF14 (GTIN-14 olarak da bilinir)
TÜR_UPC_A
TÜR_UPC_E
TÜR_MSI
TÜR_MSI_KONTROL_TUTAM
TÜR_POSTNET
TÜR_GEZEGEN
TÜR_RMS4CC
TÜR_KIX
TÜR_IMB
TÜR_KODABAR
TÜR_KODU_11
TÜR_İLAÇ_KODU
TÜR_İLAÇ_KODU_İKİ_PARÇA
Desteklenen tüm barkod türleri için örnek görsellere bakın

PNG ve JPG görselleri hakkında bir not
PNG veya JPG resimleri kullanmak istiyorsanız, Imagick veya GD kütüphanesini yüklemeniz gerekir . Bu paket, yüklüyse Imagick'i kullanır veya GD'ye geri döner. Her ikisini de yüklediyseniz ancak belirli bir yöntem istiyorsanız, tercihinizi zorlamak için $renderer->useGd()veya kullanabilirsiniz.$renderer->useImagick()

Örnekler
HTML'de gömülü PNG resmi
$barcode = (new Picqer\Barcode\Types\TypeCode128())->getBarcode('081231723897');
$renderer = new Picqer\Barcode\Renderers\PngRenderer();
echo '<img src="data:image/png;base64,' . base64_encode($renderer->render($barcode, $barcode->getWidth() * 2)) . '">';
JPG barkodunu diske kaydet
$barcode = (new Picqer\Barcode\Types\TypeCodabar())->getBarcode('081231723897');
$renderer = new Picqer\Barcode\Renderers\JpgRenderer();

file_put_contents('barcode.jpg', $renderer->render($barcode, $barcode->getWidth() * 2));
Tek satırlık SVG çıktısını diske aktarma
file_put_contents('barcode.svg', (new Picqer\Barcode\Renderers\SvgRenderer())->render((new Picqer\Barcode\Types\TypeKix())->getBarcode('6825ME601')));
v3'e yükseltme
v2'den v3'e yükseltme yaparken hiçbir şeyi değiştirmenize gerek yok. Yukarıda bu kütüphaneyi v3'ten beri kullanmanın yeni tercih edilen yolunu bulabilirsiniz. Ancak eski stil hala işe yarıyor.

Renderer'lara aynı arayüzü vermek için, renkleri ayarlamak artık her zaman bir RGB renk dizisiyle yapılır. Eski BarcodeGenerator* sınıflarını kullanırsanız ve adları ('kırmızı') veya hex kodları (#3399ef) olan renkler kullanırsanız, bunlar ColorHelper kullanılarak dönüştürülecektir. Tüm hex kodları desteklenir, ancak renk adları için yalnızca temel renkler desteklenir.

Yeni stile geçmek istiyorsanız, işte bir örnek:

// Old style
$generator = new Picqer\Barcode\BarcodeGeneratorSVG();
echo $generator->getBarcode('081231723897', $generator::TYPE_CODE_128);

// New style
$barcode = (new Picqer\Barcode\Types\TypeCode128())->getBarcode('081231723897');
$renderer = new Picqer\Barcode\Renderers\SvgRenderer();
echo $renderer->render($barcode);
Oluşturucudaki genişlik artık widthFactor yerine nihai sonucun genişliğidir. Dinamik genişlikleri korumak istiyorsanız, kodlanmış Barkodun genişliğini alabilir ve onu widthFactor ile çarparak daha öncekiyle aynı sonucu elde edebilirsiniz. Burada widthFactor'ın 2'ye eşit olduğu bir örneğe bakın:

// Old style
$generator = new Picqer\Barcode\BarcodeGeneratorSVG();
echo $generator->getBarcode('081231723897', $generator::TYPE_CODE_128, 2. 30);

// New style
$barcode = (new Picqer\Barcode\Types\TypeCode128())->getBarcode('081231723897');
$renderer = new Picqer\Barcode\Renderers\SvgRenderer();
echo $renderer->render($barcode, $barcode->getWidth() * 2, 30);
Önceki stil jeneratörleri
Sürüm 3'te barkod tipi kodlayıcılar ve görüntü işleyiciler tamamen ayrıdır. Bu, kendi işleyicinizi oluşturmayı çok daha kolay hale getirir. Eski yöntem "üreticiler" kullanmaktı. Aşağıda, v3'te de hala çalışan bu üreteçlerin eski örnekleri bulunmaktadır.

Kullanım
İstediğiniz çıktı için barkod üretecini başlatın, sonra ->getBarcode() rutinini istediğiniz kadar çağırın.

<?php
require 'vendor/autoload.php';

// This will output the barcode as HTML output to display in the browser
$generator = new Picqer\Barcode\BarcodeGeneratorHTML();
echo $generator->getBarcode('081231723897', $generator::TYPE_CODE_128);
Bu güzellikle sonuçlanacak:
Barkod 081231723897, Kod 128 olarak

Yöntem getBarcode()aşağıdaki parametreleri kabul eder:

$barcodeBarkodda kodlanması gereken dize
$typeBarkod türü, sınıfta tanımlanan sabitleri kullanın
$widthFactorGenişlik, verilerin uzunluğuna bağlıdır; bu faktörle barkod çubuklarını varsayılandan daha geniş yapabilirsiniz
$heightBarkodun toplam yüksekliği piksel cinsinden
$foregroundColorÇubukların renklerinin (ön plan rengi) RGB dizisi veya dizesi olarak onaltılık kod
Tüm parametrelerin kullanımına örnek:

<?php

require 'vendor/autoload.php';

$redColor = [255, 0, 0];

$generator = new Picqer\Barcode\BarcodeGeneratorPNG();
file_put_contents('barcode.png', $generator->getBarcode('081231723897', $generator::TYPE_CODE_128, 3, 50, $redColor));
Görüntü türleri
$generatorSVG = new Picqer\Barcode\BarcodeGeneratorSVG(); // Vector based SVG
$generatorPNG = new Picqer\Barcode\BarcodeGeneratorPNG(); // Pixel based PNG
$generatorJPG = new Picqer\Barcode\BarcodeGeneratorJPG(); // Pixel based JPG
$generatorHTML = new Picqer\Barcode\BarcodeGeneratorHTML(); // Pixel based HTML
$generatorHTML = new Picqer\Barcode\BarcodeGeneratorDynamicHTML(); // Vector based HTML
HTML'de gömülü PNG resmi
$generator = new Picqer\Barcode\BarcodeGeneratorPNG();
echo '<img src="data:image/png;base64,' . base64_encode($generator->getBarcode('081231723897', $generator::TYPE_CODE_128)) . '">';
JPG barkodunu diske kaydet
$generator = new Picqer\Barcode\BarcodeGeneratorJPG();
file_put_contents('barcode.jpg', $generator->getBarcode('081231723897', $generator::TYPE_CODABAR));
Tek satırlık SVG çıktısını diske aktarma
file_put_contents('barcode.svg', (new Picqer\Barcode\BarcodeGeneratorSVG())->getBarcode('6825ME601', Picqer\Barcode\BarcodeGeneratorSVG::TYPE_KIX));

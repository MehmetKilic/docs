# Y�kseltme Rehberi

- [4.1'den 4.2'ye Y�kseltme](#upgrade-4.2)
- [4.0'dan 4.1'e Y�kseltme](#upgrade-4.1)

<a name="upgrade-4.2"></a>
## 4.1'den 4.2'ye Y�kseltme

### PHP 5.4+

Laravel 4.2, PHP 5.4.0 veya daha �st�n� gerektirir.

### Modellerdeki Soft Silmeler Art�k Trait Kullan�yorlar

Modellerde soft silmeler kullan�yorsan�z, `softDeletes` propertisi ��kart�lm��t�r. Art�k a�a��dakine benzer �ekilde `SoftDeletingTrait` kullanmal�s�n�z:

	use Illuminate\Database\Eloquent\SoftDeletingTrait;

	class User extends Eloquent {
		use SoftDeletingTrait;
	}

Ayr�ca, `dates` propertisine `deleted_at` s�tununu elle eklemeniz gerekir:

	class User extends Eloquent {
		use SoftDeletingTrait;

		protected $dates = ['deleted_at'];
	}

T�m soft silme i�lemlerinin API'si ayn� kalm��t�r.

### View / Pagination Environment S�n�flar�n�n Ad� De�i�ti

�ayet `Illuminate\View\Environment` s�n�f�n� veya `Illuminate\Pagination\Environment` s�n�f�n� do�rudan referans ediyorsan�z, kodunuzu bunlar yerine `Illuminate\View\Factory` ve `Illuminate\Pagination\Factory` s�n�flar�n� referans verecek �ekilde g�ncellemelisiniz. Bu iki s�n�f�n isimleri, i�levlerini daha iyi yans�tmas� i�in de�i�tirilmi�tir.

<a name="upgrade-4.1"></a>
## 4.0'dan 4.1'e Y�kseltme

### Composer Ba��ml�l���n�n Y�kseltilmesi

Uygulaman�z� Laravel 4.1'e y�kseltmek i�in, `composer.json` dosyan�zdaki `laravel/framework` s�r�m�n� `4.1.*` olarak de�i�tirin.

### Dosyalar�n De�i�tirilmesi

Uygulaman�zdaki `public/index.php` dosyas�n� [ambardaki bu yeni kopya](https://github.com/laravel/laravel/blob/master/public/index.php) ile de�i�tirin.

Uygulaman�zdaki `artisan` dosyas�n� [ambardaki bu yeni kopya](https://github.com/laravel/laravel/blob/master/artisan) ile de�i�tirin.

### Yap�land�rma Dosya ve Se�eneklerinin Eklenmesi

Uygulaman�zdaki `app/config/app.php` yap�land�rma dosyan�zdaki `aliases` ve `providers` dizilerini g�ncelleyin. Bu dizilerin g�ncellenmi� de�erleri [bu dosyada](https://github.com/laravel/laravel/blob/master/app/config/app.php) bulunabilir. Kendi �zel ve paket servis sa�lay�c�lar�n� / aliaslar� tekrar eklemeyi unutmay�n.

[Ambardaki](https://github.com/laravel/laravel/blob/master/app/config/remote.php) yeni `app/config/remote.php` dosyas�n� ekleyin.

Uygulaman�zdaki `app/config/session.php` dosyan�za yeni `expire_on_close` yap�land�rma se�ene�ini ekleyin. �n tan�ml� de�er `false` olmal�d�r.

Uygulaman�zdaki `app/config/queue.php` dosyan�za yeni `failed` yap�land�rma kesimini ekleyin. Bu kesimin default de�erleri ��yledir:

	'failed' => array(
		'database' => 'mysql', 'table' => 'failed_jobs',
	),

**(�ste�e Ba�l�)** Uygulaman�zdaki `app/config/view.php` dosyan�zdaki `pagination` yap�land�rma se�ene�ini `pagination::slider-3` olarak g�ncelleyin.

### Controller G�ncellemeleri

E�er `app/controllers/BaseController.php` dosyas�nda en �stte bir `use` c�mlesi varsa, buradaki `use Illuminate\Routing\Controllers\Controller;` olan yeri `use Illuminate\Routing\Controller;` olarak g�ncelleyin.

### Password Reminders G�ncellemeleri

�ifre hat�rlat�c�lar� daha b�y�k esneklik olmas� i�in elden ge�irilmi�tir. Artisan `php artisan auth:reminders-controller` komutunu �al��t�rmak suretiyle yeni iskelet controlleri inceleyebilirsiniz. Ayr�ca [g�ncellenmi� dok�mantasyonu](/docs/security#password-reminders-and-reset) da okuyabilir ve uygulaman�z� ona g�re g�ncelleyebilirsiniz.

Uygulaman�zdaki `app/lang/en/reminders.php` dil dosyas�n� [g�ncellenen bu dosyaya](https://github.com/laravel/laravel/blob/master/app/lang/en/reminders.php) uyacak �ekilde g�ncelleyin.

### Ortam Saptama G�ncellemeleri

G�venlik sebepleri nedeniyle, uygulama ortam�n�z� tespit etmek i�in URL domainleri art�k kullan�lmayabilir. Bu de�erler kolayl�kla kafeslenebilir ve sald�rganlar�n bir istek i�in ortam� modifiye etmesine imkan verebilir. Ortam tespitinizi makine host adlar� (Mac & Ubuntu �zerinde `hostname` komutu) kullanacak �ekilde de�i�tirmelisiniz.

### Daha Sade ve Basit G�nl�k Dosyalar�

Laravel art�k tek bir log dosyas� �retir: `app/storage/logs/laravel.log`. Bununla birlikte, bu davran��� yine de `app/start/global.php` dosyan�zda yap�land�rabilirsiniz.

### En Sonda B�l� Varsa Yeniden Y�nlendirin ��kart�lmas�

Uygulaman�z�n `bootstrap/start.php` dosyas�ndan `$app->redirectIfTrailingSlash()` �a�r�s�n� ��kart�n. Bu i�levsellik �imdi frameworkle gelen `.htaccess` dosyas� taraf�ndan halledildi�i i�in bu metod art�k gerekli de�ildir.

Sonra da, sizin Apache `.htaccess` dosyan�z�n yerine, sondaki b�l�leri halleden [bu yenisini](https://github.com/laravel/laravel/blob/master/public/.htaccess) koyun.

### G�ncel Rotaya Eri�im

G�ncel rotaya `Route::getCurrentRoute()` yerine �imdi `Route::current()` ile eri�ilmektedir.

### Composer G�ncellemesi

Yukar�daki de�i�iklikleri tamamlad�ktan sonra, �ekirdek application dosyalar�n� g�ncellemek i�in `composer update` fonksiyonunu �al��t�rabilirsiniz! E�er s�n�f y�kleme (class load) hatalar� al�rsan�z, `update` komutunu �u �ekilde etkinle�tirilmi� `--no-scripts` se�ene�i ile kullanmay� deneyin: `composer update --no-scripts`.

### Joker Olay Dinleyiciler

Joker Olay Dinleyiciler art�k handler fonksiyon parametrelerinize event'i eklemez. �ayet ate�lenen olay� bulman�z gerekiyorsa, `Event::firing()` kullanmal�s�n�z.
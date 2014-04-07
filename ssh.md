# SSH

- [Yap�land�rma](#configuration)
- [Temel Kullan�m](#basic-usage)
- [G�revler](#tasks)
- [SFTP Dosya �ndirmeleri](#sftp-downloads)
- [SFTP Dosya G�ndermeleri](#sftp-uploads)
- [Uzak G�nl�klerin �zlenmesi](#tailing-remote-logs)
- [Envoy G�rev �al��t�r�c�s�](#envoy-task-runner)

<a name="configuration"></a>
## Yap�land�rma

Laravel uzak sunuculara SSH (Secure Shell) ileti�imi ve komutlar �al��t�rmak i�in basit bir yol i�erir ve uzak sunucularda �al��an Artisan g�revlerini kolayca in�a etmenize imkan verir. `SSH` facade'� uzak sunucular�n�za ba�lanman�z ve komutlar �al��t�rman�z i�in eri�im noktas� sa�lar.

Yap�land�rma dosyas� `app/config/remote.php` konumundad�r ve uzak ba�lant�lar�n�z� yap�land�rmak i�in gerekli t�m se�enekleri i�erir. Bu dosyadaki `connections` dizisi, s�r�c�lerinizin isimlerine g�re anahtarlanm�� bir listesini ta��r. Bu `connections` dizisindeki host, username, password, key gibi eri�im g�ven bilgilerini (credentials) doldurduktan sonra uzak g�revleri �al��t�rmaya haz�r olacaks�n�z. Unutmay�n, `SSH`, ya bir password ya da bir SSH key kullanarak kimlik do�rulamas� yapabilmektedir.

> **Not:** Uzak sunucunuzda �e�itli g�revleri kolayca �al��t�rma ihtiyac�n�z m� var? [Envoy g�rev �al��t�r�c�s�na](#envoy-task-runner) bir bak�n!

<a name="basic-usage"></a>
## Temel Kullan�m

#### Komutlar� Default Sunucuda �al��t�rmak

Komutlar�n�z� `default` uzak ba�lant�n�zda �al��t�rmak i�in `SSH::run` metodunu kullan�n:

	SSH::run(array(
		'cd /var/www',
		'git pull origin master',
	));

#### Komutlar� Belirli Bir Ba�lant�da �al��t�rmak

Alternatif olarak, `into` metodunu kullanmak suretiyle komutlar� belirli bir ba�lant� �zerinde �al��t�rabilirsiniz:

	SSH::into('staging')->run(array(
		'cd /var/www',
		'git pull origin master',
	));

#### Komut ��kt�lar�n� Yakalamak

`run` metoduna bir Closure ge�mek suretiyle, uzak komutlar�n�z�n "canl�" ��kt�s�n� yakalayabilirsiniz:

	SSH::run($commands, function($line)
	{
		echo $line.PHP_EOL;
	});

## G�revler
<a name="tasks"></a>

E�er her zaman birlikte �al��mas� gereken bir grup komut tan�mlaman�z gerekiyorsa, bir `task` (g�rev) tan�mlamak i�in `define` metodunu kullanabilirsiniz:

	SSH::into('staging')->define('deploy', array(
		'cd /var/www',
		'git pull origin master',
		'php artisan migrate',
	));

Bu �ekilde bir task tan�mlad�ktan sonra, onu �al��t�rmak i�in `task` metodunu kullanabilirsiniz:

	SSH::into('staging')->task('deploy', function($line)
	{
		echo $line.PHP_EOL;
	});

<a name="sftp-downloads"></a>
## SFTP Dosya �ndirmeleri

`SSH` s�n�f� `get` ve `getString` metodlar� kullan�larak, dosyalar indirmek i�in basit bir yol sa�lar:

	SSH::into('staging')->get($remotePath, $localPath);

	$contents = SSH::into('staging')->getString($remotePath);

<a name="sftp-uploads"></a>
## SFTP Dosya G�ndermeleri

`SSH` s�n�f� ayn� zamanda `put` ve `putString` metodlar� kullan�larak sunucuya dosyalar, hatta stringler upload etmek i�in de basit bir yol i�erir:

	SSH::into('staging')->put($localFile, $remotePath);

	SSH::into('staging')->putString($remotePath, 'Falan');

<a name="tailing-remote-logs"></a>
## Uzak G�nl�klerin �zlenmesi

Laravel sizin uzak ba�lant�lar�n�z�n herhangi birindeki `laravel.log` dosyalar�n�n izlenmesi i�in yararl� bir komut i�ermektedir. Bunun i�in basit�e `tail` Artisan komutunu kullan�n ve izlemek istedi�iniz uzak ba�lant�n�n ad�n� belirtin:

	php artisan tail staging

	php artisan tail staging --path=/path/to/log.file

<a name="envoy-task-runner"></a>
## Envoy G�rev �al��t�r�c�s�

- [Y�kleme](#envoy-installation)
- [G�revlerin �al��t�r�lmas�](#envoy-running-tasks)
- [Birden �ok Sunucu](#envoy-multiple-servers)
- [Paralel �al��t�rma](#envoy-parallel-execution)
- [Task Makrolar�](#envoy-task-macros)
- [Bildirimler](#envoy-notifications)
- [Envoy'in G�ncellenmesi](#envoy-updating-envoy)

Laravel Envoy, uzak sunucular�n�zda ortak g�revler tan�mlanmas� i�in temiz, minimal bir s�zdizimi sa�lar. [Blade](/docs/templates#blade-templating) tarz� bir s�zdizimi kullanarak yay�mlama, Artisan komutlar� ve ba�ka �eyler i�in kolayca g�revler in�a edebilirsiniz.

> **Not:** Envoy, PHP versiyon 5.4 veya daha �st�n� gerektirir ve sadece Mac / Linux i�letim sistemlerinde �al���r.

<a name="envoy-installation"></a>
### Y�kleme

�lk olarak Envoy [Phar ar�ivini](https://github.com/laravel/envoy/raw/master/envoy.phar) indirin ve eri�im kolayl��� i�in onu `envoy` olarak `/usr/local/bin` konumuna koyun. G�revleri �al��t�rabilmeniz i�in, bu `envoy` dosyas�na �al��t�rma izinleri vermeniz gerekebilir.

Sonra da, projenizin k�k�nde bir `Envoy.blade.php` dosyas� olu�turun. ��te ba�layabilece�iniz bir �rnek:

	@servers(['web' => '192.168.1.1'])

	@task('falan', ['on' => 'web'])
		ls -la
	@endtask

G�rebilece�iniz gibi, dosyan�n en �st�nde bir `@servers` dizisi tan�mlan�r. Bu sunucular�, task (g�rev) deklarasyonlar�n�z�n `on` se�ene�inde refere edebilirsiniz. Bu `@task` deklarasyonlar�n�z�n i�erisine, task �al��t�r�ld��� zaman sunucunuzda �al��t�r�lacak olan Bash kodunu koyacaks�n�z.

Bir iskelet Envoy dosyas�n� kolayca olu�turmak i�in `init` komutu kullan�labilir:

	envoy init user@192.168.1.1

<a name="envoy-running-tasks"></a>
### G�revlerin �al��t�r�lmas�

Bir g�revi �al��t�rmak i�in Envoy y�klemenizin `run` komutunu kullan�n:

	envoy run falan

E�er gerekliyse, komut sat�r� se�eneklerini kullanarak Envoy dosyas�na de�i�kenler ge�ebilirsiniz:

	envoy run deploy --branch=master

Bu se�enekleri, kulland���n�z Blade s�zdizimi arac�l���yla kullanabilirsiniz:

	@servers(['web' => '192.168.1.1'])

	@task('deploy', ['on' => 'web'])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

#### Bootstrapping

Envoy dosyas�n�n i�inde de�i�kenler deklare etmek ve genel PHP i�i yapmak i�in ```@setup``` direktifini kullanabilirsiniz:

	@setup
		$now = new DateTime();

		$environment = isset($env) ? $env : "testing";
	@endsetup

Ayr�ca bir PHP dosyas� include etmek i�in ```@include``` kullanabilirsiniz:

	@include('vendor/autoload.php');

<a name="envoy-multiple-servers"></a>
### Birden �ok Sunucu

Bir g�revi birden �ok sunucuda kolayl�kla �al��t�rabilirsiniz. Sadece task deklarasyonunda sunucular� listeleyin:

	@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

	@task('deploy', ['on' => ['web-1', 'web-2']])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

�n tan�ml� olarak, ilgili g�rev her bir sunucuda seri olarak �al��t�r�lacakt�r. Yani, g�rev bir sonraki sunucuda �al��maya ba�lamadan �nce, �nceki �al��mas�n� tamamlayacakt�r.

<a name="envoy-parallel-execution"></a>
### Paralel �al��t�rma

E�er bir g�revi birden �ok sunucuda paralel olarak �al��t�rmak istiyorsan�z, yapman�z gereken tek �ey task deklarasyonunuza `parallel` se�ene�ini eklemektir:

	@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

	@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

<a name="envoy-task-macros"></a>
### Task Makrolar�

Makrolar basit bir komut kullanarak s�ral� bir bi�imde �al��acak bir g�rev k�mesi tan�mlaman�za imkan verirler. �rne�in:

	@servers(['web' => '192.168.1.1'])

	@macro('deploy')
		falan
		filan
	@endmacro

	@task('falan')
		echo "MERHABA"
	@endtask

	@task('filan')
		echo "D�NYA"
	@endtask

Art�k bu `deploy` makrosu tek, basit bir komut arac�l��� ile �al��t�r�labilecektir:

	envoy run deploy

<a name="envoy-notifications"></a>
<a name="envoy-hipchat-notifications"></a>
### Bildirimler

#### HipChat

Bir g�revi �al��t�rd�ktan sonra, basit `@hipchat` direktifini kullanarak ekibinizin HipChat odas�na bir bildirim g�nderebilirsiniz:

	@servers(['web' => '192.168.1.1'])

	@task('falan', ['on' => 'web'])
		ls -la
	@endtask

	@after
		@hipchat('token', 'room', 'Envoy')
	@endafter

Ayr�ca hipchat odas�na, �zel bir mesaj da belirtebilirsiniz. ```@setup``` i�inde deklare edilen veya ```@include``` ile dahil edilen her de�i�kenin mesajda kullan�lmas� m�mk�nd�r:

	@after
		@hipchat('token', 'room', 'Envoy', "$task ran on [$environment]")
	@endafter

Bu, ekibinizi sunucu �zerinde �al��t�r�lan g�revler hakk�nda haberdar tutmak i�in inan�lmaz basit bir yoludur.

#### Slack

[Slack'a](https://slack.com) bir bildirim g�ndermek i�in a�a��daki s�zdizimi kullan�labilir:

	@after
		@slack('team', 'token', 'channel')
	@endafter

<a name="envoy-updating-envoy"></a>
### Envoy'un G�ncellenmesi

Envoy'u g�ncellemek i�in, tek yapaca��n�z `self-update` komutunu �al��t�rmakt�r:

	envoy self-update

E�er Envoy y�kledi�iniz yer `/usr/local/bin` ise, `sudo` kullanman�z gerekebilir:

	sudo envoy self-update
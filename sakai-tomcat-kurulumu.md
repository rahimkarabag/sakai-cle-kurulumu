## Tomcat 8 Kurulumu (CentOS, RHEL, yada Fedora)

Packages dizinize geçtikten sonra tomcat dosylarını wget ile sunucumuza indiriyoruz.

```
cd /root/packages/
```
İndirdiğimiz tomcat dosyasını tar komutu ile açıp daha sonra opt dizininin altına tomcat ismiyle taşıyoruz.
```
wget http://archive.apache.org/dist/tomcat/tomcat-8/v8.0.32/bin/apache-tomcat-8.0.32.tar.gz
tar xpfz apache-tomcat-8.0.32.tar.gz
mv apache-tomcat-8.0.32 /opt/
cd /opt
ln -s apache-tomcat-8.0.32/ tomcat
```
Tomcat kurulumu için çevresel değişkenleri .bashrc dosyasını nano ile açıp içine export ifadelerini ekliyoruz. Bu sayede sunucu her başladığında tomcat için çevresel değişkenler aktif olacaktır.
```
nano /root/.bashrc
export CATALINA_HOME=/opt/tomcat
export PATH=$PATH:$CATALINA_HOME/bin
```
Sisteme restart atmak yerine source komutu ile .bashrc dosyasının içeriğini sisteme tanıtmış oluyoruz.
```
source /root/.bashrc
```
Her bir tomcat nodu bin dizini içinde setenv.sh dosyası ile farklı özellliklerle çalıştırılabilir.
```
cd /opt/tomcat/bin
nano setenv.sh
```
Bu ayarlar sizin sunucu konfigürasyonuna göre değişiklik gösterecektir. Örneğin burada tomcat için max 1024 mb hafıza ayarlanmıştır, sakai 1024 mb'tan fazla hafıza kullanmak istediğinde out of memory(yetersiz hafıza) uyarısı verecektir.
```
export JAVA_OPTS='-server -Xms512m -Xmx1024m -XX:PermSize=128m -XX:NewSize=192m -XX:MaxNewSize=384m -Djava.awt.headless=true -Dhttp.agent=Sakai -Dorg.apache.jasper.compiler.Parser.STRICT_QUOTE_ESCAPING=false -Dsun.lang.ClassLoader.allowArraySyntax=true -Djava.security.egd=file:/dev/./urandom -Duser.language=tr 
-Duser.region=TR'
```
Setenv.sh dosyasını çalıştırabilir hale getirip. Tomcat server'ın UTF8 desteğini ekliyoruz.
```
chmod a+x *.sh
cd ../conf
nano server.xml
```

server.xml dosyasının önceli hali
```
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```
server.xml dosyasının sonraki hali

```
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" URIEncoding="UTF-8"/>
```

Sakai'nin sağlıklı çalışması için tomcat dosyalarında varsayılan olarak yüklenen dosyaları webapps dizininden siliyoruz. Burada dosyalar tomcat'i yönetmek ve özelliklerini göstermek için tomcat ile beraber gelmiştir.
```
cd ..
rm -rf webapps/*
nano conf/catalina.properties
```
Sakai'nin tomcat için birkaç özel ayarı bulunmaktadır. Bunların catalina.properties dosyasında düzenlenmesi gerekir.
Düzenlenmeniz gereken satırlar aşğıda verilmiştir.

common.loader="${catalina.base}/lib","${catalina.base}/lib/*.jar","${catalina.home}/lib","${catalina.home}/lib/*.jar"   satırını aşağıdaki gibi düzenliyoruz.

```
common.loader="${catalina.base}/lib","${catalina.base}/lib/*.jar","${catalina.home}/lib","${catalina.home}/lib/*.jar","${catalina.base}/common/classes/","${catalina.base}/common/lib/*.jar"
```

server.loader= burasıda normalde boş gelmektedir, aşağıdaki satırı ekliyoruz.
```
server.loader=${catalina.base}/server/classes/,${catalina.base}/server/lib/*.jar
```

shared.loader= burası normalde boş gelmektedir, aşağıdaki satırı ekliyoruz.
```
shared.loader=${catalina.base}/shared/classes/,${catalina.base}/shared/lib/*.jar
```

Burda Sakai'nin önerdiği dizin yapısını oluşturuyoruz. Bu dizinlerin oluşturulması zorunlu değildir.
```
mkdir -p shared/classes shared/lib common/classes common/lib server/classes server/lib
```
### Tomcat'in hızlı başlaması...

```
nano /opt/tomcat/conf/context.xml
```
Ekle
```
<Context>
    <JarScanner>
        <JarScanFilter defaultPluggabilityScan="false" />
    </JarScanner>
</Context>
```


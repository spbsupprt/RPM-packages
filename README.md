# RPM-packages
RPM-packages for Otus


- создать свой RPM (можно взять свое приложение, либо собрать к примеру Apache с определенными опциями);
- cоздать свой репозиторий и разместить там ранее собранный RPM;
- реализовать это все либо в Vagrant, либо развернуть у себя через Nginx и дать ссылку на репозиторий.

# создать свой RPM

Дано:


![image](https://github.com/user-attachments/assets/b25705ab-48c8-4073-b20e-1c2df4a684a7)


Установка доп пакетов:

yum install -y wget rpmdevtools rpm-build createrepo yum-utils cmake gcc git nano

будем собирать модуль ngx_broli для Nginx, для начала качаем исходники и ставим все зависимости для сборки.

mkdir rpm && cd rpm

yumdownloader --source nginx

rpm -Uvh nginx*.src.rpm

yum-builddep nginx

качаем исходный код модуля ngx_brotli и собираем его через cmake :

git clone --recurse-submodules -j8 https://github.com/google/ngx_brotli

cd ngx_brotli/deps/brotli

mkdir out && cd out

cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DCMAKE_C_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_CXX_FLAGS="-Ofast -m64 -march=native -mtune=native -flto -funroll-loops -ffunction-sections -fdata-sections -Wl,--gc-sections" -DCMAKE_INSTALL_PREFIX=./installed ..

cmake --build . --config Release -j 2 --target brotlienc

cd ../../../..


изменияем rpmbuild/SPECS/nginx.spec , добавляем модуль и собираем пакет.

![image](https://github.com/user-attachments/assets/34bc1fcd-e70b-470d-97f6-9aa4a734c9c7)

cd ~/rpmbuild/SPECS/

rpmbuild -ba nginx.spec -D 'debug_package %{nil}'


![image](https://github.com/user-attachments/assets/540b32c6-2a4b-476d-87c2-b854daa2827e)


![image](https://github.com/user-attachments/assets/f8de990c-eb7d-48cf-80a8-5367330b9d6b)


Копируем пакеты в общий каталог и устанавливаем:


cp ~/rpmbuild/RPMS/noarch/* ~/rpmbuild/RPMS/x86_64/

cd ~/rpmbuild/RPMS/x86_64

yum localinstall *.rpm -y
 
systemctl start nginx

systemctl status nginx

![image](https://github.com/user-attachments/assets/ddb7f574-4d20-44e6-8173-c27645941329)


# Создать свой репозиторий и разместить там ранее собранный RPM

mkdir /usr/share/nginx/html/repo

cp ~/rpmbuild/RPMS/x86_64/*.rpm /usr/share/nginx/html/repo/

createrepo /usr/share/nginx/html/repo/

![image](https://github.com/user-attachments/assets/bb0e4331-9f3a-4e42-ab54-48543a2b055b)

vim /etc/nginx/nginx.conf


![image](https://github.com/user-attachments/assets/e0522fc6-6971-4ab7-853b-7b433d808b88)


При проверке:

![image](https://github.com/user-attachments/assets/69114cac-8dc6-4407-85f3-18811eb39c6d)


Добавляем его в /etc/yum.repos.d

![image](https://github.com/user-attachments/assets/f83289a3-ada7-4792-a266-290bfcf4a430)



Добавим пакет в наш репозиторий:

wget https://repo.percona.com/yum/percona-release-latest.noarch.rpm

Обновим список пакетов в репозитории:

createrepo /usr/share/nginx/html/repo/

yum makecache

![image](https://github.com/user-attachments/assets/863f83d4-fdc8-4085-b606-1be58dd11cc2)


Итог, репы подключены и новые пакеты:

![image](https://github.com/user-attachments/assets/62936a5e-010f-4cf9-8c7d-003a7f2a3656)



























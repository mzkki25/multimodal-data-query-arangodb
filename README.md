# Multimodal Data Query using ArangoDB

Berikut langkah-langkah yang lebih terstruktur untuk mengatur ArangoDB dan menjalankan query di dalamnya:

### 1. Instalasi Docker Desktop (khusus untuk pengguna Windows)
1. Unduh dan instalasi Docker Desktop dari situs resminya.
2. Pastikan Docker Desktop telah terinstal dan berjalan di sistem Teman-teman.

### 2. Menjalankan ArangoDB Container
```
docker run -p 8529:8529 -e ARANGO_ROOT_PASSWORD=openSesame arangodb/enterprise:3.12.0.2
```
Ini menjalankan container ArangoDB Enterprise versi 3.12.0.2 dengan mengikuti parameter yang ditentukan.

### 3. Akses ArangoDB melalui Docker Desktop
1. Buka Docker Desktop.
2. Klik pada container ArangoDB yang sedang berjalan.
3. Akses melalui link yang tersedia untuk mengelola ArangoDB di localhost.

### 4. Masuk dan Mengautentikasi
Setelah terhubung ke ArangoDB melalui browser, masukkan kredensial pengguna dan kata sandi.
- User: root
- Password: openSesame

### 5. Menambahkan File ke dalam Container ArangoDB
1. File scores.json dapat diunduh [disini](https://version.helsinki.fi/chzhang/cikm-2020-hands-on-session-for-multi-model-queries/-/blob/master/scores.json])
2. File Person.csv, Order.json, dan KnowsGraph.csv dapat diunduh [disini](https://version.helsinki.fi/chzhang/cikm-2020-hands-on-session-for-multi-model-queries/-/tree/master/Multi-model-data)
3. Unduh dan simpan file-file tersebut.
4. Masukkan file-file tersebut ke dalam folder usr di dalam image container ArangoDB.
5. Baca dokumentasi untuk mengetahui cara menambahkan file ke dalam container ArangoDB.

### 6. Menjalankan Perintah untuk Mengimpor Data
```
usr/bin/arangoimport --file usr/scores.json --collection "scores" --server.username root --create-collection true
```
```
usr/bin/arangoimport --file usr/Person.csv --type csv --translate "id=_key" --collection "Person" --server.username root --create-collection true
```
```
usr/bin/arangoimport --file usr/Order.json --type json --translate "id=_key" --collection "Order" --server.username root --create-collection true
```
```
usr/bin/arangoimport --file usr/KnowsGraph.csv --type csv --translate "from=_from" --translate "to=_to" --collection "KnowsGraph" --from-collection-prefix Person --to-collection-prefix Person --server.username root --create-collection true --create-collection-type edge
```
Perintah ini akan mengimpor data dari file ke dalam koleksi yang sesuai di dalam database ArangoDB.

### 7. Akses Server dan Jalankan Query
1. Buka server ArangoDB melalui browser.
2. Masuk ke bagian query untuk menjalankan query yang diberikan.

### 8. Menjalankan Query
1. Jalankan beberapa query yang terdapat pada [link ini](https://version.helsinki.fi/chzhang/cikm-2020-hands-on-session-for-multi-model-queries/-/blob/master/hands-on.ipynb)
2. Untuk soal nomor terakhir, berikut adalah query yang dapat Teman-teman jalankan:
  - Q1
    ```
    Let plist= Flatten(For order in Order return order.Orderline[*].productId)
    For item in plist
        collect productId=item with count into ct
        sort ct DESC LIMIT 10
    return {productId, ct}
    ```
  - Q2
    ```
    Return SUM(
        For person in Person For order in Order
        Filter person._key==order.PersonId and person.gender=='female' and order.OrderDate> '2008'
        return order.TotalPrice)
    ```
  - Q3
    ```
    Return count(
        For person in Any 'Person/2199023262543' KnowsGraph 
        For order in Order
        Filter person._key==order.PersonId and order.OrderDate > '2009'
        return order._key)
    ```
  - Q4
    ```
    LET shortestPath=
      (FOR vertex, edge
        IN ANY SHORTEST_PATH
        'Person/2199023259756' TO 'Person/26388279077535'
        KnowsGraph
        return vertex)
        
    LET plist=
      Flatten(
              For item in shortestPath
              For order in Order
              Filter item._key==order.PersonId
              return order.Orderline
              )
              
    For item in plist
    collect productId=item.productId with count into ct
    Sort ct DESC limit 5
    return {productId, ct}
    ```
  - Q5
    ```
    Let plist=(
        For order in Order
        collect PersonId=order.PersonId into group
        sort SUM(group[*].order.TotalPrice) DESC LIMIT 2
        return {PersonId, Monetary:SUM(group[*].order.TotalPrice)}
      )
      
    Let set1=(
        For vertex in 0..3 outbound CONCAT("Person/", plist[0].PersonId) KnowsGraph OPTIONS{ bfs: TRUE}
        return vertex
      )
      
    Let set2=(
        For vertex in 0..3 outbound CONCAT("Person/", plist[1].PersonId) KnowsGraph OPTIONS{bfs: TRUE}
        return vertex
    )
    
    return count(intersection(set1,set2))
    ```
Teman-teman dapat menyalin dan menjalankan masing-masing query pada editor query di ArangoDB untuk mendapatkan hasilnya. Pastikan Teman-teman memahami tujuan dan output yang diharapkan dari setiap query sebelum menjalankannya. Teman-teman dapat mempelajari dokumentasinya di [link ini](https://version.helsinki.fi/chzhang/cikm-2020-hands-on-session-for-multi-model-queries/-/blob/master/hands-on.ipynb)

# Belajar Docker untuk Pemula - Membuat TODO App

TODO app ini adalah contoh app untuk mendemokan proses membuat aplikasi dengan Docker, terdiri dari:
- Frontend dengan [Vue JS framework](https://docs.vuejs.id/v2/guide/)
- Backend dengan [Golang](https://dasarpemrogramangolang.novalagung.com/)
- Database dengan Postgres

Semua komponen dipackage dengan docker

## Menjalankan Postgres dengan Docker

Step 1: Jalankan Postgres
```bash
docker run -p 5432:5432 --name todo-postgres -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=rahasia -e POSTGRES_DB=belajar -v $(pwd)/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql -d postgres
```

Step 2: Export konfigurasi database sebagai environment variabel
```bash
export DB_HOST=localhost
export DB_PORT=5432
export DB_USER=postgres
export DB_PASSWORD=rahasia
export DB_DATABASE=belajar
```

Step 3: Jalankan Backend Golang app di lokal komputer
```bash
go run backend/main.go
```

Step 4: Jalankan Frontend JS app di lokal komputer
```bash
cd frontend
yarn install
yarn serve
```

Step 5: Buka browser untuk mulai mengakses app TODO
```bash
http://localhost:8081
```

Step 6: Cek input TODO tersimpan di database
```bash
docker exec -it todo-postgres psql -U postgres -W belajar
SELECT * FROM todo;
```

## Menjalankan Semuanya sebagai Docker Container

Step 1: Buat Docker Network
```bash
docker network create todo
```

Step 2: Jalankan Docker postgres di dalam network
```bash
docker run -d \
-p 5432:5432 \
--name todo-postgres \
-e POSTGRES_USER=postgres \
-e POSTGRES_PASSWORD=rahasia \
-e POSTGRES_DB=belajar \
-v $(pwd)/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql \
--network todo \
postgres
```

Step 3: Buat docker image untuk backend
```bash
cd backend
docker build -t todo-backend:v1 .
```

Step 4: Jalankan backend sebagai docker container di dalam network
```bash
docker run -d \
-p 8080:8080 \
--name todo-backend \
-e DB_USER=postgres \
-e DB_PASSWORD=rahasia \
-e DB_HOST=todo-postgres \
-e DB_PORT=5432 \
-e DB_DATABASE=belajar \
--network todo \
todo-backend:v1
```

Step 5: Buat docker image untuk frontend
```bash
cd frontend
docker build -t todo-frontend:v1 .
```

Step 6: Jalankan frontend sebagai docker container
```bash
docker run -d \
-p 8081:8080 \
--name todo-frontend \
--network todo \
todo-frontend:v1
```

## Menjalankan Container menggunakan Docker Compose

Step 1: Buat file docker-compose.yaml

Step 2: Masukkan instruksi untuk menjalankan semua container
```
version: '3'
services:
    todo-postgres:
        image: postgres
        ports:
            - 5432:5432
        environment:
            - POSTGRES_USER=postgres
            - POSTGRES_PASSWORD=rahasia
            - POSTGRES_DB=belajar
        volumes:
            - ./postgres/init.sql:/docker-entrypoint-initdb.d/init.sql
    todo-backend:
        build: ./backend
        ports:
            - 8080:8080
        environment:
            - DB_USER=postgres
            - DB_PASSWORD=rahasia
            - DB_HOST=todo-postgres
            - DB_PORT=5432
            - DB_DATABASE=belajar
        depends_on:
            - todo-postgres
    todo-frontend:
        build: ./frontend
        ports:
            - 8081:8080
```

Step 3: Jalankan docker compose
```
docker compose up
```
# Snippetbox

> Paste and share snippets of text

[![forthebadge](https://forthebadge.com/images/badges/built-with-love.svg)](https://forthebadge.com) [![forthebadge](https://forthebadge.com/images/badges/made-with-go.svg)](https://forthebadge.com)

## Setup

### 1. Create a Docker container for MySQL
```bash
docker run -d --name snippetbox-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password mysql
```

### 2. Connect to the MySQL container
```bash
docker exec -it snippetbox-mysql /bin/bash
```

### 3. Connect to the MySQL server
```bash
mysql -u root -p
```

### 4. Create the database
```sql
CREATE DATABASE snippetbox CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

USE snippetbox;

CREATE TABLE snippets (
    id INTEGER NOT NULL PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(100) NOT NULL,
    content TEXT NOT NULL,
    created DATETIME NOT NULL,
    expires DATETIME NOT NULL
);

CREATE INDEX idx_snippets_created ON snippets(created);

CREATE TABLE sessions (
    token CHAR(43) PRIMARY KEY,
    data BLOB NOT NULL,
    expiry TIMESTAMP(6) NOT NULL
);

CREATE INDEX sessions_expiry_idx ON sessions (expiry);

CREATE TABLE users (
    id INTEGER NOT NULL PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    hashed_password CHAR(60) NOT NULL,
    created DATETIME NOT NULL
);

ALTER TABLE users ADD CONSTRAINT users_uc_email UNIQUE (email);
```

### 5. Seed the database
```sql
INSERT INTO snippets (title, content, created, expires) VALUES (
    'An old silent pond',
    'An old silent pond...\nA frog jumps into the pond,\nsplash! Silence again.\n\n– Matsuo Bashō',
    UTC_TIMESTAMP(),
    DATE_ADD(UTC_TIMESTAMP(), INTERVAL 365 DAY)
);

INSERT INTO snippets (title, content, created, expires) VALUES (
    'Over the wintry forest',
    'Over the wintry\nforest, winds howl in rage\nwith no leaves to blow.\n\n– Natsume Soseki',
    UTC_TIMESTAMP(),
    DATE_ADD(UTC_TIMESTAMP(), INTERVAL 365 DAY)
);

INSERT INTO snippets (title, content, created, expires) VALUES (
    'First autumn morning',
    'First autumn morning\nthe mirror I stare into\nshows my father''s face.\n\n– Murakami Kijo',
    UTC_TIMESTAMP(),
    DATE_ADD(UTC_TIMESTAMP(), INTERVAL 7 DAY)
);
```

### 6. Create a new user
Replace localhost with the IP address of the Docker container (172.17.0.1 in my case)
```sql
CREATE USER 'web'@'172.17.0.1';

GRANT SELECT, INSERT, UPDATE, DELETE ON snippetbox.* TO 'web'@'172.17.0.1';

ALTER USER 'web'@'172.17.0.1' IDENTIFIED BY 'password';

FLUSH PRIVILEGES;
```

### 7. Generate a self-signed certificate
Go installation directory may vary depending on your OS
```bash
go run /usr/local/go/src/crypto/tls/generate_cert.go --rsa-bits=2048 --host=localhost

mkdir tls
mv cert.pem tls/cert.pem
mv key.pem tls/key.pem
```

## Run tests

### 1. Create the test database
```sql
CREATE DATABASE test_snippetbox CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE USER 'test_web'@'172.17.0.1';

GRANT CREATE, DROP, ALTER, INDEX, SELECT, INSERT, UPDATE, DELETE ON test_snippetbox.* TO 'test_web'@'172.17.0.1';

ALTER USER 'test_web'@'172.17.0.1' IDENTIFIED BY 'pass';
```

### 2. Run all tests
```bash
go test -v ./...
```

### 3. Run a specific test
```bash
go test -v -run="^TestUserSignup$" ./cmd/web/
```
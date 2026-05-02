# Anslut till DuckLake från Java

En steg-för-steg tutorial för att ansluta till DuckLake via Java på en virtuell Linux-server (Ubuntu) på KTH Cloud.

OBS — Linux-distribution: Det går att välja olika Linux-distributioner när du skapar en deployment på KTH Cloud. Det här tutorialet är skrivet för Ubuntu och använder apt som pakethanterare. Använder du en annan distro (t.ex. Fedora, Debian, Arch) kan kommandona för att installera paket se annorlunda ut.

## Förutsättningar

- En SSH-nyckel skapad och uppladdad till din profil på [KTH Cloud](https://cloud.cbh.kth.se)
- Credentials från [DuckLake Access Manager](https://ducklake-access-manager.app.cloud.cbh.kth.se/)

---

## Steg 1 — Anslut till servern

Öppna PowerShell och kör:

```bash
ssh <din deployment>@deploy.cloud.cbh.kth.se
```

---

## Steg 2 — Byt till bash-shell

```bash
bash
```

---

## Steg 3 — Installera Java (om det inte redan finns)

```bash
sudo apt update
sudo apt install -y default-jdk
```

Verifiera installationen:

```bash
java -version
```

---

## Steg 4 — Skapa arbetsmapp och ladda ner DuckDB JDBC

```bash
mkdir ~/ducklake-java && cd ~/ducklake-java
```

Kolla senaste tillgängliga version:

```bash
curl -s https://repo1.maven.org/maven2/org/duckdb/duckdb_jdbc/maven-metadata.xml | grep "<version>"
```

Ladda ner senaste versionen (byt ut versionsnumret om det finns en nyare):

```bash
curl -L -o duckdb.jar https://repo1.maven.org/maven2/org/duckdb/duckdb_jdbc/1.5.2.1/duckdb_jdbc-1.5.2.1.jar
```

---

## Steg 5 — Installera nano

```bash
sudo apt install -y nano
```

---

## Steg 6 — Skapa .env-fil med dina credentials

Hämta dina credentials från [DuckLake Access Manager](https://ducklake-access-manager.app.cloud.cbh.kth.se/) och skapa filen:

```bash
nano -w .env
```

Klistra in och fyll i dina värden:

```bash
# Storage (S3/Garage)
AWS_ACCESS_KEY_ID=<DIN_ACCESS_KEY_ID>
AWS_SECRET_ACCESS_KEY=<DIN_SECRET_ACCESS_KEY>
AWS_ENDPOINT_URL=<DIN_ENDPOINT_URL>
AWS_DEFAULT_REGION=<DIN_REGION>
S3_BUCKET=<DITT_BUCKET_NAMN>

# Catalog (Postgres)
PGHOST=<DIN_POSTGRES_HOST>
PGPORT=5432
PGDATABASE=<DITT_DATABAS_NAMN>
PGUSER=<DITT_ANVÄNDARNAMN>
PGPASSWORD=<DITT_LÖSENORD>
```

Spara med `Ctrl+O` → `Enter` → `Ctrl+X`.

---

## Steg 7 — Skapa Java-filen

```bash
nano -w <filensnamn>.java
```

Klistra in koden från filen `filensnamn.java` i detta repo.

Spara med `Ctrl+O` → `Enter` → `Ctrl+X`.

---

## Steg 8 — Ladda miljövariabler och kör

Ladda in `.env`-filen i shellet:

```bash
export $(cat .env | grep -v '^#' | grep -v '^\s*$' | xargs)
```

Kompilera och kör:

```bash
javac -cp duckdb.jar <filensnamn>.java && java -cp .:duckdb.jar <filensnamn>
```

---

## Felsökning

| Problem | Lösning |
|---|---|
| `Permission denied (publickey)` | Din SSH-nyckel är inte uppladdad på KTH Cloud |
| `Miljövariabler saknas` | Kör `export`-kommandot i Steg 8 innan du kör Java |
| Kompileringsfel med långa rader | Använd `nano -w` för att stänga av automatisk radbrytning |
| Filen försvann efter omstart | Lägg till persistent storage i deploymentet på KTH Cloud |

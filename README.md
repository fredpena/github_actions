```yml
name: Java CI with Maven # Asigna un nombre a la acción. En este caso, se llama "Java CI with Maven".

on:
  push:
    branches: [ main ] # Ejecuta el workflow cuando se realiza un push a la rama "main".
  pull_request:
    branches: [ main ] # Ejecuta el workflow cuando se abre un pull request hacia la rama "main".

jobs:
  build:
    runs-on: ubuntu-latest # Especifica el sistema operativo del runner que se utilizará. En este caso, "ubuntu-latest" utilizará la última versión disponible de Ubuntu.

    steps:
      - uses: actions/checkout@v4 # Utiliza la acción de GitHub para hacer checkout del repositorio en el runner.

      - name: Set up JDK 17 # Nombre del paso.
        uses: actions/setup-java@v3 # Utiliza la acción de GitHub para configurar la versión de Java.
        with:
          java-version: '17' # Especifica que se debe configurar Java 17.
          distribution: 'temurin' # Especifica que se debe utilizar la distribución de Java "temurin".
          cache: maven # Habilita el caché para las dependencias de Maven, lo que puede acelerar las construcciones futuras.

      - name: Build with Maven # Nombre del paso.
        run: mvn -B package --file pom.xml # Ejecuta el comando Maven para empaquetar la aplicación en modo batch (sin interacción).

      - name: Run tests # Nombre del paso.
        run: mvn test # Ejecuta los tests utilizando Maven.
```

# Validación de Código

## Formateo y Linting

Automatiza la validación del estilo de tu código para asegurarte de que cumple con las convenciones establecidas.

### Ejemplo: Usar Checkstyle para Java

```yml
name: Java Lint with Checkstyle

on: [ push, pull_request ]

jobs:
  lint:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven

      - name: Install Checkstyle
        run: |
          wget -O checkstyle.jar https://github.com/checkstyle/checkstyle/releases/download/checkstyle-8.42/checkstyle-8.42-all.jar

      - name: Run Checkstyle
        run: |
          java -jar checkstyle.jar -c /google_checks.xml src/main/java
```

# Pruebas de Seguridad

## Análisis de Dependencias

Automatiza el escaneo de dependencias para identificar vulnerabilidades conocidas.

### Ejemplo: Usar OWASP Dependency-Check

```yml
name: OWASP Dependency-Check

on: [ push, pull_request ]

jobs:
  dependency-check:
    runs-on: ubuntu-latest

    steps:
      - name: Install OWASP Dependency-Check
        run: |
          wget -O dependency-check.tar.gz https://github.com/jeremylong/DependencyCheck/releases/download/v6.1.6/dependency-check-6.1.6-release.zip
          tar -xvf dependency-check.tar.gz

      - name: Run OWASP Dependency-Check
        run: |
          ./dependency-check/bin/dependency-check.sh --project "my-java-app" --scan ./ --format "HTML"
      - name: Upload dependency-check report
        uses: actions/upload-artifact@v2
        with:
          name: dependency-check-report
          path: dependency-check-report.html
```

# Generación de Documentación

## Generar Javadoc

Automatiza la generación de documentación Javadoc y súbela como artefacto.

### Ejemplo: Generar y Publicar Javadoc

```yml
name: Generate Javadoc

on: [ push, pull_request ]

jobs:
  javadoc:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven

      - name: Generate Javadoc
        run: mvn javadoc:javadoc

      - name: Upload Javadoc
        uses: actions/upload-artifact@v2
        with:
          name: javadoc
          path: target/site/apidocs
```

# Pruebas Unitarias con Cobertura

## Cobertura de Pruebas

Automatiza la ejecución de pruebas unitarias y mide la cobertura de código.

### Ejemplo: Usar JaCoCo para Cobertura

```yml
name: Java CI with Maven and JaCoCo

on: [ push, pull_request ]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven

      - name: Cache Maven packages
        uses: actions/cache@v2
        with:
          path: ~/.m2/repository
          key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
          restore-keys: ${{ runner.os }}-maven

      - name: Build with Maven
        run: mvn -B package --file pom.xml

      - name: Run tests and generate coverage report
        run: mvn test jacoco:report

      - name: Upload coverage report
        uses: actions/upload-artifact@v2
        with:
          name: jacoco-report
          path: target/site/jacoco
```

# Análisis de Calidad de Código

## Integración con SonarQube

Aunque SonarQube es un servicio de terceros, puedes realizar análisis de calidad de código de manera similar usando
herramientas locales.

### Ejemplo: Usar SpotBugs

```yml
name: SpotBugs Analysis

on: [ push, pull_request ]

jobs:
  spotbugs:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven

      - name: Install SpotBugs
        run: |
          wget -O spotbugs.zip https://repo.maven.apache.org/maven2/com/github/spotbugs/spotbugs/4.2.3/spotbugs-4.2.3.zip
          unzip spotbugs.zip

      - name: Run SpotBugs
        run: |
          ./spotbugs-4.2.3/bin/spotbugs -textui -effort:max -high src/main/java

      - name: Upload SpotBugs report
        uses: actions/upload-artifact@v2
        with:
          name: spotbugs-report
          path: spotbugs-4.2.3

```

# Realizar Comprobaciones de Seguridad

## Escaneo de Seguridad de Código Fuente

Puedes utilizar herramientas como CodeQL para realizar análisis de seguridad en el código fuente.

### Ejemplo: Usar CodeQL

```yml
name: CodeQL Analysis

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  analyze:
    name: Analyze
    runs-on: ubuntu-latest

    steps:

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v1
        with:
          languages: java

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v1
```

# Automatizar Merges

## Auto-Merge de Pull Requests

Puedes configurar GitHub Actions para realizar merges automáticos de pull requests cuando todos los checks pasen
exitosamente.

### Ejemplo: Auto-Merge

```yml
name: Auto-Merge

on:
  pull_request:
    types: [ closed ]

jobs:
  auto-merge:
    runs-on: ubuntu-latest
    if: github.event.pull_request.merged == true
    steps:

      - name: Setup Git
        run: |
          git config --global user.name 'github-actions[bot]'
          git config --global user.email 'github-actions[bot]@users.noreply.github.com'

      - name: Merge main into dev
        run: |
          git checkout dev
          git merge main
          git push origin dev
```
# Construa-o-Clone-do-TradeMap-para-Acompanhar-a-Cotação-das-Ações-da-B3-com-Spring-Boot-e-Kotlin

Para construir um aplicativo semelhante ao TradeMap, que acompanha cotações de ações da B3 em tempo real e permite aos usuários adicionar ativos favoritos, vamos dividir o projeto em duas partes principais: uma API RESTful usando Spring Boot com Kotlin e um aplicativo Android nativo usando Kotlin.

### 1. API RESTful com Spring Boot e Kotlin

#### Estrutura do Projeto em Módulos

Vamos organizar o projeto em módulos para separar as responsabilidades e facilitar a manutenção e escalabilidade:

- **core**: Módulo principal com as entidades, serviços e repositórios.
- **api**: Módulo que expõe os endpoints REST para o aplicativo Android.
- **kafka**: Módulo para integração com Apache Kafka, responsável pelo envio de atualizações em tempo real para o aplicativo Android.

#### Tecnologias Utilizadas

- **Spring Boot**: Para criação da API RESTful.
- **Kotlin**: Linguagem de programação para o desenvolvimento tanto da API quanto do aplicativo Android.
- **Apache Kafka**: Para transmitir as atualizações de cotações em tempo real para o aplicativo Android.

#### Exemplo de Código

Aqui está um exemplo simplificado de como você pode estruturar a API RESTful usando Spring Boot e Kotlin:

**1. Dependências do Gradle para o módulo `api` e configuração básica do Spring Boot:**

```kotlin
// build.gradle.kts (módulo api)
plugins {
    id("org.springframework.boot") version "2.5.3"
    id("io.spring.dependency-management") version "1.0.11.RELEASE"
    kotlin("jvm") version "1.5.21"
    kotlin("plugin.spring") version "1.5.21"
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
    runtimeOnly("com.h2database:h2")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}

application {
    mainClassName = "com.example.trademap.TradeMapApplication"
}
```

**2. Entidades Kotlin para representar ações e favoritos:**

```kotlin
// src/main/kotlin/com/example/trademap/core/entity/Stock.kt
package com.example.trademap.core.entity

import javax.persistence.Entity
import javax.persistence.GeneratedValue
import javax.persistence.GenerationType
import javax.persistence.Id

@Entity
data class Stock(
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0,
    val symbol: String,
    val companyName: String,
    val currentPrice: Double
)

// src/main/kotlin/com/example/trademap/core/entity/Favorite.kt
package com.example.trademap.core.entity

import javax.persistence.Entity
import javax.persistence.GeneratedValue
import javax.persistence.GenerationType
import javax.persistence.Id

@Entity
data class Favorite(
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long = 0,
    val userId: Long,
    val stockId: Long
)
```

**3. Repositórios Kotlin usando Spring Data JPA:**

```kotlin
// src/main/kotlin/com/example/trademap/core/repository/StockRepository.kt
package com.example.trademap.core.repository

import com.example.trademap.core.entity.Stock
import org.springframework.data.jpa.repository.JpaRepository
import org.springframework.stereotype.Repository

@Repository
interface StockRepository : JpaRepository<Stock, Long> {
    fun findBySymbol(symbol: String): Stock?
}

// src/main/kotlin/com/example/trademap/core/repository/FavoriteRepository.kt
package com.example.trademap.core.repository

import com.example.trademap.core.entity.Favorite
import org.springframework.data.jpa.repository.JpaRepository
import org.springframework.stereotype.Repository

@Repository
interface FavoriteRepository : JpaRepository<Favorite, Long> {
    fun findByUserId(userId: Long): List<Favorite>
}
```

**4. Serviços Kotlin para lógica de negócio:**

```kotlin
// src/main/kotlin/com/example/trademap/core/service/StockService.kt
package com.example.trademap.core.service

import com.example.trademap.core.entity.Stock
import com.example.trademap.core.repository.StockRepository
import org.springframework.stereotype.Service

@Service
class StockService(private val stockRepository: StockRepository) {

    fun getStockBySymbol(symbol: String): Stock? {
        return stockRepository.findBySymbol(symbol)
    }

    fun getAllStocks(): List<Stock> {
        return stockRepository.findAll()
    }
}
```

**5. Controladores Kotlin para definir os endpoints REST:**

```kotlin
// src/main/kotlin/com/example/trademap/api/controller/StockController.kt
package com.example.trademap.api.controller

import com.example.trademap.core.entity.Stock
import com.example.trademap.core.service.StockService
import org.springframework.web.bind.annotation.*

@RestController
@RequestMapping("/api/stocks")
class StockController(private val stockService: StockService) {

    @GetMapping("/{symbol}")
    fun getStockBySymbol(@PathVariable symbol: String): Stock? {
        return stockService.getStockBySymbol(symbol)
    }

    @GetMapping
    fun getAllStocks(): List<Stock> {
        return stockService.getAllStocks()
    }
}
```

#### Integrando com Apache Kafka

Para integrar o Apache Kafka na sua aplicação Spring Boot, você pode usar o módulo `kafka`. Configurações e beans para produção e consumo de mensagens podem ser configurados neste módulo.

### 2. Aplicativo Android Nativo com Kotlin

#### Desenvolvimento do Aplicativo

- Utilize o Android Studio para criar um novo projeto em Kotlin.
- Implemente telas para visualizar cotações de ações e adicionar/remover favoritos.
- Integre com a API RESTful criada usando Retrofit para consumir os endpoints.

#### Funcionalidades do Aplicativo

- Tela inicial com lista de ações e seus preços atualizados.
- Tela de detalhes para exibir mais informações sobre uma ação específica.
- Funcionalidade para adicionar ações aos favoritos e visualizar a lista de favoritos.

#### Exemplo de Código Android

Um exemplo simples de como você pode consumir a API RESTful no Android usando Retrofit:

```kotlin
// Exemplo de interface Retrofit para a API RESTful
interface StockApiService {

    @GET("/api/stocks/{symbol}")
    suspend fun getStockBySymbol(@Path("symbol") symbol: String): Response<Stock>

    @GET("/api/stocks")
    suspend fun getAllStocks(): Response<List<Stock>>
}

// Exemplo de uso no Android
val retrofit = Retrofit.Builder()
    .baseUrl("http://localhost:8080")  // Substitua pelo seu endpoint real
    .addConverterFactory(GsonConverterFactory.create())
    .build()

val service = retrofit.create(StockApiService::class.java)

// Chamadas assíncronas usando coroutines (Kotlin suspending functions)
lifecycleScope.launch {
    val response = service.getAllStocks()
    if (response.isSuccessful) {
        val stocks = response.body()
        // Faça algo com os dados recebidos
    } else {
        // Trate o erro de requisição
    }
}
```

### Considerações Finais

Este projeto envolve a criação de uma arquitetura completa, desde a API RESTful até o aplicativo Android nativo, utilizando tecnologias modernas como Spring Boot, Kotlin e Apache Kafka para garantir uma experiência robusta e em tempo real aos usuários. A organização em módulos facilita a manutenção e o desenvolvimento paralelo das diferentes partes do sistema.

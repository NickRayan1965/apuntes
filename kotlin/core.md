# Kotlin

## Agregar Dependencias
build.gradle.kts (Module:App).
Luego clic en el botón "Sync Now"
```kts
dependencies {
    // ... tus dependencias por defecto de Compose ...

    // Retrofit (Para la Capa de Red)
    implementation("com.squareup.retrofit2:retrofit:2.11.0")
    implementation("com.squareup.retrofit2:converter-gson:2.11.0")

    // Coil (Para cargar imágenes en la Capa de UI más adelante)
    implementation("io.coil-kt:coil-compose:2.6.0")
}
```
## Permisos de Internet
AndroidManifest.xml
Lo agregamos dentro de la etiqueta `<manifest>` pero fuera de `<application>`
```xml
<uses-permission android:name="android.permission.INTERNET" />
```

### Crear Data class
@SerializedName es una anotación de la biblioteca Gson que se utiliza para mapear los nombres de las propiedades de un objeto JSON a los nombres de las propiedades de una clase en Kotlin.
```kotlin
data class User(
    @SerializedName("id") val id: Int,
    @SerializedName("name") val name: String,
    @SerializedName("username") val username: String,
)
```

### Definir la Interfaz del API (Retrofit)
- @GET("character"): Indica que se hará una petición HTTP GET al endpoint relativo character.
- suspend: Esta palabra clave de Kotlin indica que la función es una Corrutina.
```kotlin
package com.example.rickandmortyapp.data

import retrofit2.http.GET

interface RickAndMortyApiService {
    
    @GET("character")
    suspend fun getCharacters(): CharacterResponse
}
```

### Instancia de Retrofit
- object: crea un singleton
- by lazy: inicializa la propiedad solo cuando se accede por primera vez, lo que mejora el rendimiento y evita la creación innecesaria de objetos.
```kotlin
package com.example.rickandmortyapp.data.remote

import retrofit2.Retrofit
import retrofit2.converter.gson:GsonConverterFactory

object RetrofitClient {
    private const val BASE_URL = "https://rickandmortyapi.com/api/"

    val apiService: RickAndMortyApiService by lazy {
        Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(RickAndMortyApiService::class.java)
    }
}
```
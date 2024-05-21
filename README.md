## SafeNet Library


SafeNet is a library engineered to streamline the configuration of the network layer and dependency injection in your projects.
It leverages annotations to automatically furnish essential components such as repositories, API services, and use cases. 
This eliminates the need for manual setup using @Module and @Provides annotations, thereby simplifying the process of establishing the Network Layer and Dependency Injection for network layer.


In the SafeNet library, the ```@ServiceModule```, ```@RepositoryModule```, and ```@UseCaseModule``` annotations are used to automate the process of dependency injection. These annotations are used to mark classes that should be provided as dependencies in your project.

The ```@Authenticated``` annotation is used to add an access token to the route,  ```@CustomHeaders``` annotation to add a custom headers to the route
and the ```@BaseUrl``` annotation is used to specify the base URL for the API service.


## Installation

```
dependencies {

    implementation("com.thesubgraph:safenet:$latest-version")
    implementation("com.thesubgraph:annotations:$latest-version")
    kapt("com.thesubgraph.processor:$latest-version")

  //other Dependency
}
```
Replace ```latest-version ``` with the latest version of the library.


## Here's a basic example of how you might use these annotations in your project:

```

// Annotate your ApiService
@ServiceModule
@BaseUrl("BASE_URL")
//@CustomHeaders(key values) if you want to custom headers add for all routes in this microservices
interface MyApiService : WebServiceInterface {
    @GET("photos")
    @Authenticated
    //@CustomHeaders(key values) if you want add
    suspend fun getPhotos(
        @Query("page") page: Int,
        @Query("per_page") perSize: Int,
    ): Response<List<PhotoDto>>

}


@ServiceModule
@BaseUrl("ANOTHER_BASE_URL")
//@CustomHeaders(key values) if you want to custom headers add for all routes in this microservices
interface MyAnotherService : WebServiceInterface {
    @GET("endpoint")
    @Authenticated
    suspend fun getData(
        @Query("page") page: Int,
        @Query("per_page") perSize: Int,
    ): Response<ResponseBody>

}

```
## Repository
```

interface MyRepository {
    fun getPhotos(page: Int, pageSize: Int): Flow<ValueResult<List<Photo>>>
}

// Annotate your Repository
@RepositoryModule(MyRepository::class)
class MyRepositoryImpl @Inject constructor(
    private val apiService: ApiService, 
    private val requestWrapper: RequestWrapper,
) : MyRepository {
override fun getPhotos(page: Int, pageSize: Int): Flow<ValueResult<List<Photo>>> {
        return flow {
            val result = requestWrapper.execute(mapper = { photos ->
                photos.map { it.mapToDomain() } }) {
                apiService.getPhotos(page = page, perSize = pageSize)
            }
            emit(result)
        }
    }
}
```
## UseCase 

```
// Annotate your UseCase
@UseCaseModule
class FetchDataUseCase(private val repository: MyRepository) {
    // Use the injected Repository here
}

```

## DI Module

```
@Module(includes = [RetrofitModule::class, AppModule::class,DispatcherModule::class])
@InstallIn(SingletonComponent::class)
object Modules

```
```includes = [RetrofitModule::class, AppModule::class, DispatcherModule::class]:```
This parameter in the ```@Module``` annotation tells Dagger that the Modules object includes the ```RetrofitModule```, ```AppModule```, and ```DispatcherModule```. This means that the dependencies provided by these modules are also available for injection wherever the Modules object is used.

In this example, the MyApiService is annotated with ```@ServiceModule```, which means it will be provided as a dependency where needed. 
The MyRepositoryImpl is annotated with ```@RepositoryModule(MyRepository::class)```, indicating that it requires an instance of MyApiService to be injected. 
The FetchDataUseCase is annotated with ```@UseCaseModule```, indicating that it should be provided as a dependency where needed. 




# Android Kotlin coroutine support

The Authgear Android SDK comes with kotlin support out of the box. In fact, the SDK is written in kotlin!

If you are using kotlin, you can benefit from the suspend function APIs authgear provides. For example, the above authorize example can be written as follows:

```kotlin
import kotlinx.coroutines.*

class MyAwesomeViewModel(application: MyAwesomeApplication) : AndroidViewModel(application) {
    // Other methods

    // This is called when login button is clicked.
    fun login() {
        viewModelScope {
            withContext(Dispatchers.IO) {
                try {
                    val app = getApplication<MyAwesomeApplication>()
                    val state = app.authgear.authenticate(AuthenticateOptions(redirectUri = "com.myapp://host/path"))
                    // User is logged in!
                } catch (e: Throwable) {
                    // Something went wrong.
                }
            }
        }
    }
}
```

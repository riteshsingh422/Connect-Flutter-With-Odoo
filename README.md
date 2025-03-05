# Connect Flutter with Odoo RPC

This guide will help you integrate **Flutter** with **Odoo** using the `odoo_rpc` package and HTTP requests. We will set up authentication with Odoo and manage login using **GetX**.

## Folder Structure

Inside your Flutter project, create a folder named `services` inside the `lib` directory. Then, create a file `odoo_api.dart` inside `services`.

```
lib/
  ├── services/
  │     ├── odoo_api.dart
  ├── controllers/
  │     ├── login_controller.dart
  ├── bindings/
  │     ├── login_binding.dart
  ├── views/
  │     ├── login_view.dart
```

## 1. Odoo API Service (`odoo_api.dart`)

Create `odoo_api.dart` inside the `services` folder:

```dart
import 'package:http/http.dart' as http;
import 'dart:convert';

class OdooApi {
  final String baseUrl;

  OdooApi(this.baseUrl);

  Future<Map<String, dynamic>> authenticate(
      String database, String username, String password) async {
    try {
      final url = Uri.parse("$baseUrl/web/session/authenticate");
      final response = await http.post(
        url,
        headers: {"Content-Type": "application/json"},
        body: jsonEncode({
          "jsonrpc": "2.0",
          "method": "call",
          "params": {"db": database, "login": username, "password": password}
        }),
      );

      print("\ud83d\udce2 RAW RESPONSE: \${response.body}"); // Debug: print full response
      
      final responseData = jsonDecode(response.body);
      if (response.statusCode == 200 && responseData["result"] != null) {
        final result = responseData["result"];
        print("Authentication Response: \$result");
        if (result["uid"] == null) {
          throw Exception("User ID is null. Login failed.");
        }
        return result;
      } else {
        print("Odoo Error: \${responseData["error"]}");
        throw Exception(responseData["error"]?["message"] ?? "Invalid response");
      }
    } catch (e) {
      print("Login error: \$e");
      rethrow;
    }
  }
}
```

## 2. Login Controller (`login_controller.dart`)

Create `login_controller.dart` inside the `controllers` folder:

```dart
import 'package:chatting_app/app/services/odoo_api.dart';
import 'package:flutter/material.dart';
import 'package:get/get.dart';

class LoginController extends GetxController {
  final OdooApi _odooApi;
  var isLoading = false.obs;

  final databaseController = TextEditingController();
  final emailController = TextEditingController();
  final passwordController = TextEditingController();

  LoginController(String baseUrl) : _odooApi = OdooApi(baseUrl);

  Future<void> serverLogin(String database, String email, String password) async {
    try {
      isLoading.value = true;
      print("Authenticating: DB=\$database, User=\$email");

      final session = await _odooApi.authenticate(database, email, password);
      print("Login successful: User ID = \${session['uid']}");

      Get.snackbar('Success', 'Login successful!', snackPosition: SnackPosition.BOTTOM);
      Get.offNamed('/home');
    } catch (e) {
      print("Login failed: \$e");
      Get.snackbar('Error', 'Login failed: \${e.toString()}',
          snackPosition: SnackPosition.BOTTOM,
          backgroundColor: Colors.red,
          colorText: Colors.white);
    } finally {
      isLoading.value = false;
    }
  }

  @override
  void onClose() {
    databaseController.dispose();
    emailController.dispose();
    passwordController.dispose();
    super.onClose();
  }
}
```

## 3. Login Binding (`login_binding.dart`)

Create `login_binding.dart` inside the `bindings` folder:

```dart
import 'package:get/get.dart';
import '../controllers/login_controller.dart';

class LoginBinding extends Bindings {
  @override
  void dependencies() {
    Get.lazyPut<LoginController>(
      () => LoginController('http://your_ip_address:your_odoo_port'),
    );
  }
}
```

## 4. Login Button in Login View (`login_view.dart`)

Modify your login button inside `login_view.dart`:

```dart
Obx(
  () => ElevatedButton(
    onPressed: controller.isLoading.value
        ? null
        : () => controller.serverLogin(
              'your_database_name',
              controller.emailController.text.trim(),
              controller.passwordController.text,
            ),
    child: controller.isLoading.value
        ? CircularProgressIndicator()
        : Text("Login"),
  ),
)
```

## Final Steps

- Replace **`your_ip_address`** and **`your_odoo_port`** with your actual Odoo server details.
- Replace **`your_database_name`** with the Odoo database name.
- Run your Flutter app and test the login integration.

## 📌 Notes
- This implementation uses **GetX** for state management and navigation.
- The API service is using `http` package for making requests.
- Make sure your Odoo server is running and accessible from your Flutter app.

---
### 🚀 Happy Coding! 🎉



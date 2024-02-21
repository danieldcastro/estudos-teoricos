# Tornando o aplicativo de instância única

Aplicativos de instância única permitem apenas uma instância do aplicativo em execução por vez. Os aplicativos Flutter para Windows são multi-instânciados por padrão. Se uma primeira instância existir ao tentar iniciar uma nova instância, encerre a nova instância e ative a primeira instância.

## Por meio do método `FindWindow`

Encontre a janela `HWND` nomeada `single_instance_example` usando `FindWindow`, ative a janela se ela existir e encerre o programa atual.

> Por favor, altere `single_instance_example` para o nome do seu aplicativo.

Altere o arquivo `windows/runner/main.cpp` da seguinte maneira:

```cpp
// ...

int APIENTRY wWinMain(_In_ HINSTANCE instance, _In_opt_ HINSTANCE prev,
                      _In_ wchar_t *command_line, _In_ int show_command) {
+  HWND hwnd = ::FindWindow(L"FLUTTER_RUNNER_WIN32_WINDOW", L"single_instance_example");
+  if (hwnd != NULL) {
+    ::ShowWindow(hwnd, SW_NORMAL);
+    ::SetForegroundWindow(hwnd);
+    return EXIT_FAILURE;
+  }

  // ...
  
  if (!window.CreateAndShow(L"single_instance_example", origin, size)) {
    return EXIT_FAILURE;
  }

  // ...
}
```
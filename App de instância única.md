# Regra para Garantir Apenas uma Instância do Aplicativo Windows em Flutter

Neste guia, discutiremos como implementar uma regra para garantir que apenas uma instância do seu aplicativo Windows, desenvolvido com Flutter, seja executada de cada vez. Essa funcionalidade é útil para manter a consistência e evitar conflitos quando lidando com instâncias do aplicativo.

Isso é comumente implementado usando um mecanismo de mutex (Mutex de Sincronização), que é um objeto que permite que várias threads compartilhem recursos, garantindo que apenas uma thread por vez tenha acesso ao recurso compartilhado. Usaremos um mutex para verificar se uma instância do aplicativo já está em execução.

> Por favor, altere `single_instance_example` para o nome do seu aplicativo.

Altere o arquivo `windows/runner/main.cpp` da seguinte maneira:

## Passo 1: Definir um Nome Único para o Mutex

```cpp
#define APP_MUTEX_NAME L"UniqueAppMutex"
```

Este nome será usado para identificar exclusivamente o mutex. Certifique-se de que seja único para evitar conflitos com outros aplicativos.

## Passo 2: Criar e Verificar o Mutex

```cpp
HANDLE hMutex = ::CreateMutex(nullptr, TRUE, APP_MUTEX_NAME);
if (hMutex != nullptr && ::GetLastError() != ERROR_ALREADY_EXISTS) {
    // Se o mutex não existe, esta é a primeira instância do aplicativo
    // Prossiga com a execução normal do aplicativo.
} else {
    // Se o mutex já existe, outra instância do aplicativo está em execução
    // Você pode fazer o que for necessário aqui, como trazer a janela existente para frente, etc.
}
```

Aqui, criamos o mutex e verificamos se ele já existe. Se não existir, esta é a primeira instância do aplicativo, e o código dentro do primeiro bloco será executado. Caso contrário, se o mutex já existir, outra instância do aplicativo está em execução e o código dentro do segundo bloco será executado.

## Passo 3: Gerenciar a Instância do Aplicativo

Dentro do bloco que trata o caso de uma instância do aplicativo já estar em execução, você pode fazer ajustes necessários. Por exemplo, trazer a janela existente para a frente e sair.

```cpp
HWND hwnd = ::FindWindow(L"FLUTTER_RUNNER_WIN32_WINDOW", L"single_instance_example");
if (hwnd != nullptr) {
    // Ajuste a janela existente conforme necessário
}
return EXIT_FAILURE;
```

Este trecho de código localiza a janela existente do aplicativo e a ajusta conforme necessário.

## Passo 4: Liberar o Mutex

Certifique-se de liberar o mutex quando o aplicativo terminar.

```cpp
::ReleaseMutex(hMutex);
```

Isso deve ser feito antes de sair do aplicativo para garantir que o mutex seja liberado corretamente.

## Exemplo Completo de Implementação

```cpp
#include <windows.h>

#define APP_MUTEX_NAME L"UniqueAppMutex"

int APIENTRY wWinMain(_In_ HINSTANCE hInstance, _In_opt_ HINSTANCE hPrevInstance,
                      _In_ LPWSTR lpCmdLine, _In_ int nCmdShow) {

    HANDLE hMutex = ::CreateMutex(nullptr, TRUE, APP_MUTEX_NAME);
    if (hMutex != nullptr && ::GetLastError() != ERROR_ALREADY_EXISTS) {
        // Esta é a primeira instância do aplicativo
        // Prossiga com a execução normal do aplicativo.
    } else {
        // Outra instância do aplicativo está em execução
        // Faça os ajustes necessários, como trazer a janela existente para frente.
        HWND hwnd = ::FindWindow(L"FLUTTER_RUNNER_WIN32_WINDOW", L"single_instance_example");
        if (hwnd != nullptr) {
            // Ajuste a janela existente conforme necessário
        }
        return EXIT_FAILURE;
    }

    // O código restante do aplicativo continua aqui...

    ::ReleaseMutex(hMutex);
    return EXIT_SUCCESS;
}
```

Este código pode ser inserido no ponto de entrada (`wWinMain`) do seu aplicativo Windows para garantir que apenas uma instância do aplicativo seja executada de cada vez.

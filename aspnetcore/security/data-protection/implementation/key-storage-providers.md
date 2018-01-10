---
title: Dostawcy magazynu kluczy
author: rick-anderson
description: Dostawcy magazynu kluczy
keywords: Encryption,ASP.NET Core
ms.author: riande
manager: wpickett
ms.date: 01/14/2017
ms.topic: article
ms.assetid: 423e0a79-2f34-44c4-aaf3-146a53c39251
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/data-protection/implementation/key-storage-providers
ms.openlocfilehash: d4b286dc47f8d66e6d09c3e0f48e6326139c8e1e
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 11/10/2017
---
# <a name="key-storage-providers"></a>Dostawcy magazynu kluczy

<a name="data-protection-implementation-key-storage-providers"></a>

Domyślnie system ochrony danych [wykorzystuje heurystyki](xref:security/data-protection/configuration/default-settings) ustalenie, gdzie powinien zostać utrwalony materiał kluczy kryptograficznych. Deweloper może zastąpić heurystyki i ręcznie określ lokalizację.

> [!NOTE]
> Jeśli określisz lokalizacja jawna trwałości klucza systemu ochrony danych wyrejestrowania szyfrowanie klucza domyślne w mechanizm rest podany heurystyki, więc klucze będzie już być szyfrowane, gdy. Zaleca się że dodatkowo [mechanizm jawne klucza szyfrowania określony](key-encryption-at-rest.md#data-protection-implementation-key-encryption-at-rest-providers) przez aplikacje produkcyjne.

System ochrony danych jest dostarczany z wielu dostawców magazynu kluczy w polu.

## <a name="file-system"></a>System plików

Przewidujemy, że wiele aplikacji będzie używać repozytorium kluczy opartych na systemie plików. Aby to skonfigurować, należy wywołać [PersistKeysToFileSystem](https://github.com/aspnet/DataProtection/blob/rel/1.1.0/src/Microsoft.AspNetCore.DataProtection/DataProtectionBuilderExtensions.cs) procedury konfiguracji, jak pokazano poniżej. Podaj `DirectoryInfo` wskazujący repozytorium, w którym można przechowywać kluczy.

```csharp
sc.AddDataProtection()
       // persist keys to a specific directory
       .PersistKeysToFileSystem(new DirectoryInfo(@"c:\temp-keys\"));
   ```

`DirectoryInfo` Może wskazywać na katalog na komputerze lokalnym lub może wskazywać folderu w udziale sieciowym. Jeśli wskazuje katalog na komputerze lokalnym (oraz scenariusz jest, że tylko aplikacje na komputerze lokalnym będą musieli używać tego repozytorium), należy rozważyć użycie [interfejsu DPAPI systemu Windows](key-encryption-at-rest.md#data-protection-implementation-key-encryption-at-rest) do szyfrowania kluczy w stanie spoczynku. W przeciwnym razie należy rozważyć użycie [certyfikatu X.509](key-encryption-at-rest.md#data-protection-implementation-key-encryption-at-rest) do szyfrowania kluczy w stanie spoczynku.

## <a name="azure-and-redis"></a>Azure i pamięci podręcznej Redis

`Microsoft.AspNetCore.DataProtection.AzureStorage` i `Microsoft.AspNetCore.DataProtection.Redis` pakietów umożliwia przechowywanie kluczy ochrony danych w usłudze Azure Storage lub w pamięci podręcznej Redis. Klucze mogą być współużytkowane przez wiele wystąpień aplikacji sieci web. Aplikacji platformy ASP.NET Core mogą udostępniać pliki cookie uwierzytelniania lub ochrony CSRF na wielu serwerach. Aby skonfigurować na platformie Azure, wywoływanie jednego z [PersistKeysToAzureBlobStorage](https://github.com/aspnet/DataProtection/blob/rel/1.1.0/src/Microsoft.AspNetCore.DataProtection.AzureStorage/AzureDataProtectionBuilderExtensions.cs) overloads, jak pokazano poniżej.

```
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToAzureBlobStorage(new Uri("<blob URI including SAS token>"));

    services.AddMvc();
}
```

Zobacz też [kod testu Azure](https://github.com/aspnet/DataProtection/blob/rel/1.1.0/samples/AzureBlob/Program.cs).

Aby skonfigurować w pamięci podręcznej Redis, wywoływanie jednego z [PersistKeysToRedis](https://github.com/aspnet/DataProtection/blob/rel/1.1.0/src/Microsoft.AspNetCore.DataProtection.Redis/RedisDataProtectionBuilderExtensions.cs) overloads, jak pokazano poniżej.

```
public void ConfigureServices(IServiceCollection services)
{
    // Connect to Redis database.
    var redis = ConnectionMultiplexer.Connect("<URI>");
    services.AddDataProtection()
        .PersistKeysToRedis(redis, "DataProtection-Keys");

    services.AddMvc();
}
```

Zobacz następujące tematy, aby uzyskać więcej informacji:

- [ConnectionMultiplexer programie StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/Basics.md)
- [Pamięć podręczna Azure Redis](https://docs.microsoft.com/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache#connect-to-the-cache)
- [Kod testu redis](https://github.com/aspnet/DataProtection/blob/rel/1.1.0/samples/Redis/Program.cs).

## <a name="registry"></a>Rejestr

Czasami aplikacja może nie mieć uprawnienia do zapisu w systemie plików. Rozważmy scenariusz, w którym aplikacja działa jako konto usługi wirtualnych (takich jak tożsamość puli aplikacji w w3wp.exe). W takim przypadku administrator może przygotowana klucz rejestru, który jest odpowiedni ACLed tożsamości konta usługi. Wywołanie [PersistKeysToRegistry](https://github.com/aspnet/DataProtection/blob/rel/1.1.0/src/Microsoft.AspNetCore.DataProtection/DataProtectionBuilderExtensions.cs) procedury konfiguracji, jak pokazano poniżej. Podaj `RegistryKey` wskazuje lokalizację przechowywania kluczy kryptograficznych/wartości.

```csharp
   sc.AddDataProtection()
       // persist keys to a specific location in the system registry
       .PersistKeysToRegistry(Registry.CurrentUser.OpenSubKey(@"SOFTWARE\Sample\keys"));
   ```

Jeśli używasz rejestru systemowego jako mechanizmu stanu trwałego, rozważ użycie [interfejsu DPAPI systemu Windows](key-encryption-at-rest.md#data-protection-implementation-key-encryption-at-rest) do szyfrowania kluczy w stanie spoczynku.

## <a name="custom-key-repository"></a>Niestandardowe repozytorium klucza

Jeśli mechanizmów w polu nie są odpowiednie, deweloper można określić własne mechanizmu stanu trwałego klucza zapewniając niestandardowego `IXmlRepository`.
---
title: "Carregamentos de arquivos no núcleo do ASP.NET"
author: ardalis
description: "Como usar o modelo de associação e streaming para carregar arquivos no ASP.NET MVC de núcleo."
keywords: "ASP.NET Core, o carregamento do arquivo de modelo de associação IFormFile, streaming"
ms.author: riande
manager: wpickett
ms.date: 7/5/2017
ms.topic: article
ms.assetid: ebc98159-a028-4a94-b06c-43981c79c6be
ms.technology: aspnet
ms.prod: asp.net-core
uid: mvc/models/file-uploads
ms.openlocfilehash: 3d42fd0657bcfb4b0fdab699bbcb572e5736688c
ms.sourcegitcommit: 9cdbfd0d670d70b9c354216aabee260c52dad5ee
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/12/2017
---
# <a name="file-uploads-in-aspnet-core"></a><span data-ttu-id="c8bbc-104">Carregamentos de arquivos no núcleo do ASP.NET</span><span class="sxs-lookup"><span data-stu-id="c8bbc-104">File uploads in ASP.NET Core</span></span>

<span data-ttu-id="c8bbc-105">Por [Steve Smith](https://ardalis.com/)</span><span class="sxs-lookup"><span data-stu-id="c8bbc-105">By [Steve Smith](https://ardalis.com/)</span></span>

<span data-ttu-id="c8bbc-106">Ações do ASP.NET MVC suportam a carregamento de um ou mais arquivos usando o modelo simples de associação para os arquivos menores ou streaming para arquivos maiores.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-106">ASP.NET MVC actions support uploading of one or more files using simple model binding for smaller files or streaming for larger files.</span></span>

[<span data-ttu-id="c8bbc-107">Exibir ou baixar o exemplo do GitHub</span><span class="sxs-lookup"><span data-stu-id="c8bbc-107">View or download sample from GitHub</span></span>](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/models/file-uploads/sample/FileUploadSample)

## <a name="uploading-small-files-with-model-binding"></a><span data-ttu-id="c8bbc-108">Carregando arquivos pequenos com associação de modelo</span><span class="sxs-lookup"><span data-stu-id="c8bbc-108">Uploading small files with model binding</span></span>

<span data-ttu-id="c8bbc-109">Para carregar arquivos pequenos, você pode usar um formulário HTML várias parte ou construir uma solicitação POST usando JavaScript.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-109">To upload small files, you can use a multi-part HTML form or construct a POST request using JavaScript.</span></span> <span data-ttu-id="c8bbc-110">Um formulário de exemplo usando o Razor, que dá suporte a vários arquivos carregados, é mostrado abaixo:</span><span class="sxs-lookup"><span data-stu-id="c8bbc-110">An example form using Razor, which supports multiple uploaded files, is shown below:</span></span>

```html
<form method="post" enctype="multipart/form-data" asp-controller="UploadFiles" asp-action="Index">
    <div class="form-group">
        <div class="col-md-10">
            <p>Upload one or more files using this form:</p>
            <input type="file" name="files" multiple />
        </div>
    </div>
    <div class="form-group">
        <div class="col-md-10">
            <input type="submit" value="Upload" />
        </div>
    </div>
</form>
```

<span data-ttu-id="c8bbc-111">Para dar suporte a carregamentos de arquivos, formulários HTML devem especificar um `enctype` de `multipart/form-data`.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-111">In order to support file uploads, HTML forms must specify an `enctype` of `multipart/form-data`.</span></span> <span data-ttu-id="c8bbc-112">O `files` mostrado acima do elemento de entrada dá suporte ao carregamento de vários arquivos.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-112">The `files` input element shown above supports uploading multiple files.</span></span> <span data-ttu-id="c8bbc-113">Omitir a `multiple` atributo neste elemento de entrada para permitir que apenas um arquivo a ser carregado.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-113">Omit the `multiple` attribute on this input element to allow just a single file to be uploaded.</span></span> <span data-ttu-id="c8bbc-114">Renderiza a marcação acima em um navegador, como:</span><span class="sxs-lookup"><span data-stu-id="c8bbc-114">The above markup renders in a browser as:</span></span>

![Formulário de carregamento de arquivo](file-uploads/_static/upload-form.png)

<span data-ttu-id="c8bbc-116">O carregados no servidor de arquivos individuais podem ser acessados por meio de [associação de modelo](xref:mvc/models/model-binding) usando o [IFormFile](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.http.iformfile) interface.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-116">The individual files uploaded to the server can be accessed through [Model Binding](xref:mvc/models/model-binding) using the [IFormFile](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.http.iformfile) interface.</span></span> <span data-ttu-id="c8bbc-117">`IFormFile`tem a seguinte estrutura:</span><span class="sxs-lookup"><span data-stu-id="c8bbc-117">`IFormFile` has the following structure:</span></span>

```csharp
public interface IFormFile
{
    string ContentType { get; }
    string ContentDisposition { get; }
    IHeaderDictionary Headers { get; }
    long Length { get; }
    string Name { get; }
    string FileName { get; }
    Stream OpenReadStream();
    void CopyTo(Stream target);
    Task CopyToAsync(Stream target, CancellationToken cancellationToken = null);
}
```

> [!WARNING]
> <span data-ttu-id="c8bbc-118">Não dependa ou relação de confiança de `FileName` propriedade sem validação.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-118">Don't rely on or trust the `FileName` property without validation.</span></span> <span data-ttu-id="c8bbc-119">O `FileName` propriedade deve ser usada somente para fins de exibição.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-119">The `FileName` property should only be used for display purposes.</span></span>

<span data-ttu-id="c8bbc-120">Ao carregar arquivos usando a associação de modelo e o `IFormFile` interface, o método de ação pode aceitar um único `IFormFile` ou um `IEnumerable<IFormFile>` (ou `List<IFormFile>`) que representa vários arquivos.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-120">When uploading files using model binding and the `IFormFile` interface, the action method can accept either a single `IFormFile` or an `IEnumerable<IFormFile>` (or `List<IFormFile>`) representing several files.</span></span> <span data-ttu-id="c8bbc-121">O exemplo a seguir executa um loop em um ou mais arquivos carregados, salva-os para o sistema de arquivos local e retorna o número total e o tamanho dos arquivos carregados.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-121">The following example loops through one or more uploaded files, saves them to the local file system, and returns the total number and size of files uploaded.</span></span>

[!INCLUDE [GetTempFileName](../../includes/GetTempFileName.md)]

<span data-ttu-id="c8bbc-122">[!code-csharp[Main](file-uploads/sample/FileUploadSample/Controllers/UploadFilesController.cs?name=snippet1)]</span><span class="sxs-lookup"><span data-stu-id="c8bbc-122">[!code-csharp[Main](file-uploads/sample/FileUploadSample/Controllers/UploadFilesController.cs?name=snippet1)]</span></span>

<span data-ttu-id="c8bbc-123">Os arquivos carregados usando o `IFormFile` técnica são armazenados em buffer na memória ou no disco no servidor web antes de ser processada.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-123">Files uploaded using the `IFormFile` technique are buffered in memory or on disk on the web server before being processed.</span></span> <span data-ttu-id="c8bbc-124">Dentro do método de ação, o `IFormFile` conteúdo podem ser acessado como um fluxo.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-124">Inside the action method, the `IFormFile` contents are accessible as a stream.</span></span> <span data-ttu-id="c8bbc-125">O sistema de arquivos local, além de arquivos podem ser transmitidos para [armazenamento de BLOBs do Azure](https://azure.microsoft.com/documentation/articles/vs-storage-aspnet5-getting-started-blobs/) ou [do Entity Framework](https://docs.microsoft.com/ef/core/index).</span><span class="sxs-lookup"><span data-stu-id="c8bbc-125">In addition to the local file system, files can be streamed to [Azure Blob storage](https://azure.microsoft.com/documentation/articles/vs-storage-aspnet5-getting-started-blobs/) or [Entity Framework](https://docs.microsoft.com/ef/core/index).</span></span>

<span data-ttu-id="c8bbc-126">Para armazenar dados de arquivo binário em um banco de dados usando o Entity Framework, definir uma propriedade do tipo `byte[]` na entidade:</span><span class="sxs-lookup"><span data-stu-id="c8bbc-126">To store binary file data in a database using Entity Framework, define a property of type `byte[]` on the entity:</span></span>

```csharp
public class ApplicationUser : IdentityUser
{
    public byte[] AvatarImage { get; set; }
}
```

<span data-ttu-id="c8bbc-127">Especifique uma propriedade de viewmodel do tipo `IFormFile`:</span><span class="sxs-lookup"><span data-stu-id="c8bbc-127">Specify a viewmodel property of type `IFormFile`:</span></span>

```csharp
public class RegisterViewModel
{
    // other properties omitted

    public IFormFile AvatarImage { get; set; }
}
```

> [!NOTE]
> <span data-ttu-id="c8bbc-128">`IFormFile`pode ser usado diretamente como um parâmetro de método de ação ou como uma propriedade de viewmodel, como mostrado acima.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-128">`IFormFile` can be used directly as an action method parameter or as a viewmodel property, as shown above.</span></span>

<span data-ttu-id="c8bbc-129">Copie o `IFormFile` em um fluxo e salvá-lo para a matriz de bytes:</span><span class="sxs-lookup"><span data-stu-id="c8bbc-129">Copy the `IFormFile` to a stream and save it to the byte array:</span></span>

```csharp
// POST: /Account/Register
[HttpPost]
[AllowAnonymous]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Register(RegisterViewModel model)
{
    ViewData["ReturnUrl"] = returnUrl;
    if  (ModelState.IsValid)
    {
        var user = new ApplicationUser {
          UserName = model.Email,
          Email = model.Email
        };
        using (var memoryStream = new MemoryStream())
        {
            await model.AvatarImage.CopyToAsync(memoryStream);
            user.AvatarImage = memoryStream.ToArray();
        }
    // additional logic omitted
    
    // Don't rely on or trust the model.AvatarImage.FileName property 
    // without validation.
}
```

> [!NOTE]
> <span data-ttu-id="c8bbc-130">Tenha cuidado ao armazenar dados binários em bancos de dados relacionais, como ele pode afetar negativamente o desempenho.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-130">Use caution when storing binary data in relational databases, as it can adversely impact performance.</span></span>

## <a name="uploading-large-files-with-streaming"></a><span data-ttu-id="c8bbc-131">Carregamento de arquivos grandes de streaming</span><span class="sxs-lookup"><span data-stu-id="c8bbc-131">Uploading large files with streaming</span></span>

<span data-ttu-id="c8bbc-132">Se o tamanho ou a frequência de carregamentos de arquivos está causando problemas de recurso para o aplicativo, considere o carregamento do arquivo de streaming em vez de armazenamento em buffer em sua totalidade, assim como a abordagem de associação de modelo mostrada acima.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-132">If the size or frequency of file uploads is causing resource problems for the app, consider streaming the file upload rather than buffering it in its entirety, as the model binding approach shown above does.</span></span> <span data-ttu-id="c8bbc-133">Ao usar `IFormFile` e associação de modelo é uma quantidade solução mais simples, transmissão exige um número de etapas para implementar corretamente.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-133">While using `IFormFile` and model binding is a much simpler solution, streaming requires a number of steps to implement properly.</span></span>

> [!NOTE]
> <span data-ttu-id="c8bbc-134">Qualquer arquivo armazenado em buffer único exceder 64KB será movido de RAM para um arquivo temporário em disco no servidor.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-134">Any single buffered file exceeding 64KB will be moved from RAM to a temp file on disk on the server.</span></span> <span data-ttu-id="c8bbc-135">Os recursos (disco RAM) usados pelo carregamentos de arquivos dependem do número e tamanho dos carregamentos de arquivo simultâneas.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-135">The resources (disk, RAM) used by file uploads depend on the number and size of concurrent file uploads.</span></span> <span data-ttu-id="c8bbc-136">Fluxo não é muito sobre o desempenho, trata-se de escala.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-136">Streaming is not so much about perf, it's about scale.</span></span> <span data-ttu-id="c8bbc-137">Se você tentar carregamentos de excesso de buffer, seu site falhará quando ele ficar sem memória ou espaço em disco.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-137">If you try to buffer too many uploads, your site will crash when it runs out of memory or disk space.</span></span>

<span data-ttu-id="c8bbc-138">O exemplo a seguir demonstra como usar JavaScript/Angular para transmitir para uma ação do controlador.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-138">The following example demonstrates using JavaScript/Angular to stream to a controller action.</span></span> <span data-ttu-id="c8bbc-139">Token antiforgery do arquivo é gerado com um atributo de filtro personalizado e passada nos cabeçalhos HTTP em vez de no corpo da solicitação.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-139">The file's antiforgery token is generated using a custom filter attribute and passed in HTTP headers instead of in the request body.</span></span> <span data-ttu-id="c8bbc-140">Como o método de ação processa os dados carregados diretamente, a associação de modelo é desabilitada por outro filtro.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-140">Because the action method processes the uploaded data directly, model binding is disabled by another filter.</span></span> <span data-ttu-id="c8bbc-141">Em ação, o conteúdo do formulário é lidos usando um `MultipartReader`, que lê cada indivíduo `MultipartSection`, processar o arquivo ou armazenar o conteúdo conforme apropriado.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-141">Within the action, the form's contents are read using a `MultipartReader`, which reads each individual `MultipartSection`, processing the file or storing the contents as appropriate.</span></span> <span data-ttu-id="c8bbc-142">Depois que todas as seções foram lidos, a ação executa sua própria associação de modelo.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-142">Once all sections have been read, the action performs its own model binding.</span></span>

<span data-ttu-id="c8bbc-143">A ação inicial carrega o formulário e salva um token antiforgery em um cookie (por meio de `GenerateAntiforgeryTokenCookieForAjax` atributo):</span><span class="sxs-lookup"><span data-stu-id="c8bbc-143">The initial action loads the form and saves an antiforgery token in a cookie (via the `GenerateAntiforgeryTokenCookieForAjax` attribute):</span></span>

```csharp
[HttpGet]
[GenerateAntiforgeryTokenCookieForAjax]
public IActionResult Index()
{
    return View();
}
```

<span data-ttu-id="c8bbc-144">O atributo usa internas do ASP.NET Core [Antiforgery](xref:security/anti-request-forgery) suporte para definir um cookie com um token de solicitação:</span><span class="sxs-lookup"><span data-stu-id="c8bbc-144">The attribute uses ASP.NET Core's built-in [Antiforgery](xref:security/anti-request-forgery) support to set a cookie with a request token:</span></span>

<span data-ttu-id="c8bbc-145">[!code-csharp[Main](file-uploads/sample/FileUploadSample/Filters/GenerateAntiforgeryTokenCookieForAjaxAttribute.cs?name=snippet1)]</span><span class="sxs-lookup"><span data-stu-id="c8bbc-145">[!code-csharp[Main](file-uploads/sample/FileUploadSample/Filters/GenerateAntiforgeryTokenCookieForAjaxAttribute.cs?name=snippet1)]</span></span>

<span data-ttu-id="c8bbc-146">Angular automaticamente passa um token antiforgery em um cabeçalho de solicitação chamado `X-XSRF-TOKEN`.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-146">Angular automatically passes an antiforgery token in a request header named `X-XSRF-TOKEN`.</span></span> <span data-ttu-id="c8bbc-147">O aplicativo ASP.NET MVC de núcleo é configurado para se referir a esse cabeçalho em sua configuração no *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="c8bbc-147">The ASP.NET Core MVC app is configured to refer to this header in its configuration in *Startup.cs*:</span></span>

<span data-ttu-id="c8bbc-148">[!code-csharp[Main](file-uploads/sample/FileUploadSample/Startup.cs?name=snippet1)]</span><span class="sxs-lookup"><span data-stu-id="c8bbc-148">[!code-csharp[Main](file-uploads/sample/FileUploadSample/Startup.cs?name=snippet1)]</span></span>

<span data-ttu-id="c8bbc-149">O `DisableFormValueModelBinding` atributo, mostrado abaixo, é usado para desabilitar a associação de modelo para o `Upload` método de ação.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-149">The `DisableFormValueModelBinding` attribute, shown below, is used to disable model binding for the `Upload` action method.</span></span>

<span data-ttu-id="c8bbc-150">[!code-csharp[Main](file-uploads/sample/FileUploadSample/Filters/DisableFormValueModelBindingAttribute.cs?name=snippet1)]</span><span class="sxs-lookup"><span data-stu-id="c8bbc-150">[!code-csharp[Main](file-uploads/sample/FileUploadSample/Filters/DisableFormValueModelBindingAttribute.cs?name=snippet1)]</span></span>

<span data-ttu-id="c8bbc-151">Desde que a associação de modelo é desabilitada, o `Upload` método de ação não aceita parâmetros.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-151">Since model binding is disabled, the `Upload` action method doesn't accept parameters.</span></span> <span data-ttu-id="c8bbc-152">Ele trabalha diretamente com o `Request` propriedade `ControllerBase`.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-152">It works directly with the `Request` property of `ControllerBase`.</span></span> <span data-ttu-id="c8bbc-153">Um `MultipartReader` é usado para ler cada seção.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-153">A `MultipartReader` is used to read each section.</span></span> <span data-ttu-id="c8bbc-154">O arquivo é salvo com um nome de arquivo GUID e os dados de chave/valor são armazenados em um `KeyValueAccumulator`.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-154">The file is saved with a GUID filename and the key/value data is stored in a `KeyValueAccumulator`.</span></span> <span data-ttu-id="c8bbc-155">Depois que todas as seções tenham sido lidos, o conteúdo do `KeyValueAccumulator` são usados para associar os dados de formulário para um tipo de modelo.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-155">Once all sections have been read, the contents of the `KeyValueAccumulator` are used to bind the form data to a model type.</span></span>

<span data-ttu-id="c8bbc-156">Completo `Upload` método é mostrado abaixo:</span><span class="sxs-lookup"><span data-stu-id="c8bbc-156">The complete `Upload` method is shown below:</span></span>

[!INCLUDE [GetTempFileName](../../includes/GetTempFileName.md)]

<span data-ttu-id="c8bbc-157">[!code-csharp[Main](file-uploads/sample/FileUploadSample/Controllers/StreamingController.cs?name=snippet1)]</span><span class="sxs-lookup"><span data-stu-id="c8bbc-157">[!code-csharp[Main](file-uploads/sample/FileUploadSample/Controllers/StreamingController.cs?name=snippet1)]</span></span>

## <a name="troubleshooting"></a><span data-ttu-id="c8bbc-158">Solução de problemas</span><span class="sxs-lookup"><span data-stu-id="c8bbc-158">Troubleshooting</span></span>

<span data-ttu-id="c8bbc-159">Abaixo estão alguns problemas comuns encontrados ao trabalhar com o carregamento de arquivos e suas soluções possíveis.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-159">Below are some common problems encountered when working with uploading files and their possible solutions.</span></span>

### <a name="unexpected-not-found-error-with-iis"></a><span data-ttu-id="c8bbc-160">Erro inesperado não encontrado com o IIS</span><span class="sxs-lookup"><span data-stu-id="c8bbc-160">Unexpected Not Found error with IIS</span></span>

<span data-ttu-id="c8bbc-161">O erro a seguir indica que o carregamento do arquivo excede o servidor configurado `maxAllowedContentLength`:</span><span class="sxs-lookup"><span data-stu-id="c8bbc-161">The following error indicates your file upload exceeds the server's configured `maxAllowedContentLength`:</span></span>

```
HTTP 404.13 - Not Found
The request filtering module is configured to deny a request that exceeds the request content length.
```

<span data-ttu-id="c8bbc-162">A configuração padrão é `30000000`, que é aproximadamente 28.6 MB.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-162">The default setting is `30000000`, which is approximately 28.6MB.</span></span> <span data-ttu-id="c8bbc-163">O valor pode ser personalizado por meio da edição *Web. config*:</span><span class="sxs-lookup"><span data-stu-id="c8bbc-163">The value can be customized by editing *web.config*:</span></span>

```xml
<system.webServer>
  <security>
    <requestFiltering>
      <!-- This will handle requests up to 50MB -->
      <requestLimits maxAllowedContentLength="52428800" />
    </requestFiltering>
  </security>
</system.webServer>
```

<span data-ttu-id="c8bbc-164">Essa configuração só se aplica ao IIS.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-164">This setting only applies to IIS.</span></span> <span data-ttu-id="c8bbc-165">O comportamento não ocorre por padrão quando Kestrel de hospedagem.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-165">The behavior doesn't occur by default when hosting on Kestrel.</span></span> <span data-ttu-id="c8bbc-166">Para obter mais informações, consulte [limites de solicitações \<requestLimits\>](https://docs.microsoft.com/iis/configuration/system.webServer/security/requestFiltering/requestLimits/).</span><span class="sxs-lookup"><span data-stu-id="c8bbc-166">For more information, see [Request Limits \<requestLimits\>](https://docs.microsoft.com/iis/configuration/system.webServer/security/requestFiltering/requestLimits/).</span></span>

### <a name="null-reference-exception-with-iformfile"></a><span data-ttu-id="c8bbc-167">Exceção de referência nula com IFormFile</span><span class="sxs-lookup"><span data-stu-id="c8bbc-167">Null Reference Exception with IFormFile</span></span>

<span data-ttu-id="c8bbc-168">Se o controlador está aceitando carregar arquivos usando `IFormFile` , mas você achar que o valor é sempre nulo, confirme que o formulário HTML é especificando um `enctype` valor `multipart/form-data`.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-168">If your controller is accepting uploaded files using `IFormFile` but you find that the value is always null, confirm that your HTML form is specifying an `enctype` value of `multipart/form-data`.</span></span> <span data-ttu-id="c8bbc-169">Se esse atributo não for definido no `<form>` elemento, o carregamento de arquivo não ocorrerá e nenhum limite `IFormFile` argumentos será nulos.</span><span class="sxs-lookup"><span data-stu-id="c8bbc-169">If this attribute is not set on the `<form>` element, the file upload will not occur and any bound `IFormFile` arguments will be null.</span></span>
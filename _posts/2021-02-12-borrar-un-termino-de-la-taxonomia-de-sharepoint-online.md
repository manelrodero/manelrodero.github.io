---
layout : post
blog-width: true
title: 'Borrar un término de la taxonomía de SharePoint Online'
date: '2021-02-12 19:01:48'
published: true
tags:
- Microsoft 365
author:
  display_name: Manel Rodero
---

SharePoint Online mantiene una [taxonomía de términos](https://docs.microsoft.com/en-us/sharepoint/managed-metadata){:target="_blank"} que pueden ser utilizados por todas las _sites_ del _tenant_.

Se puede acceder a la **Term Store** para gestionar estos términos (añadir, borrar, etc.) a través de la siguiente URL:

```plaintext
https://<tenant>-admin.sharepoint.com/_layouts/15/online/AdminHome.aspx#/termStoreAdminCenter
```

Para poder realizar operaciones sobre la taxonomía es necesario asignar los permisos de [**Term Store Admin**](https://docs.microsoft.com/en-us/sharepoint/assign-roles-and-permissions-to-manage-term-sets){:target="_blank"}.

A partir de aquí, únicamente es necesario seleccionar las opciones de cada término y escoger la opción `Delete term`. Si se quiere realizar las operaciones de creación/borrado desde PowerShell se puede utilizar el módulo [**PnP PowerShell**](https://pnp.github.io/powershell/){:target="_blank"}.

La instalación es bastante sencilla desde la galería:

```powershell
Install-Module -Name PnP.PowerShell
```

A continuación hay que obtener una conexión:

```powershell
# Config Variables
$AdminCenterURL = "https://<tenant>-admin.sharepoint.com/"
 
# Connect to PnP Online
Connect-PnPOnline -Url $AdminCenterURL -Credentials (Get-Credential)
```

Si se produce un error similar al siguiente:

```powershell
Connect-PnPOnline : AADSTS65001: The user or administrator has not consented to use the application with ID
'31359c7f-bd7e-475c-86db-fdb8c937548e' named 'PnP Management Shell'. Send an interactive authorization request for
this user and resource.
Trace ID: 5a3244e2-8ebe-4b27-a5c1-499ca64a7e00
Correlation ID: 8944f2e9-17df-4cf1-a724-be1648e13ffd
Timestamp: 2021-02-12 17:59:32Z
```

significa que no se ha aprobado la ejecución de esta aplicación en el _tenant_. Para aprobarla, se tiene que realizar una primera conexión para obtener el código de autorización:

```powershell
Connect-PnPOnline -Url $AdminCenterURL -PnPManagementShell
```

Se recibirá un código que hay que introducir en [https://microsoft.com/devicelogin](https://microsoft.com/devicelogin){:target="_blank"} y después autenticarse con una cuenta que tenga el rol **Application Admin** para poder autorizar la aplicación en el _tenant_.

{: .box-note}
**Nota**: Se indicarán los permisos que requiere la aplicación y se podrá dar el consentimiento de forma individual o para toda la organización.

Las aplicación registrada se puede encontrar en el [Azure Active Directory admin center](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/AllApps){:target="_blank"} usando su nombre o su identificador de aplicación.

A partir de aquí, una vez conectados, se podrían borrar términos mediante el siguiente comando:

```powershell
# Delete the Term "South America" from Term set "regions" under "Deals Pipeline" Group
Remove-PnPTaxonomyItem "Deals Pipeline|Regions|South America" -Force
```

La lista de _cmdlets_ relacionados con la taxonomía son los siguientes:

```plaintext
CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Add-PnPTaxonomyField                               1.3.0      PnP.PowerShell
Cmdlet          Export-PnPTaxonomy                                 1.3.0      PnP.PowerShell
Cmdlet          Get-PnPTaxonomyItem                                1.3.0      PnP.PowerShell
Cmdlet          Get-PnPTaxonomySession                             1.3.0      PnP.PowerShell
Cmdlet          Import-PnPTaxonomy                                 1.3.0      PnP.PowerShell
Cmdlet          Remove-PnPTaxonomyItem                             1.3.0      PnP.PowerShell
Cmdlet          Set-PnPTaxonomyFieldValue                          1.3.0      PnP.PowerShell
```

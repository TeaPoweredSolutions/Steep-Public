# Third-party licenses and acknowledgements

The same list as the "Third-party licenses and acknowledgements" card on Steep's License page (Configure → License). Steep's license terms are in [license-terms.md](license-terms.md).

## Trademarks

Microsoft, Microsoft Endpoint Configuration Manager (MECM), Configuration Manager, CMPivot, Windows, Windows Server, Active Directory, SQL Server and Microsoft Defender are trademarks of the Microsoft group of companies ([Microsoft trademarks](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks)). Qualys and the Qualys logo are trademarks of Qualys, Inc. ([qualys.com](https://www.qualys.com/)). CVE® is a registered trademark of The MITRE Corporation. All other names and products are the property of their respective owners. Steep is made by Jonathan Dabian, doing business as Tea Powered Solutions, and is not affiliated with, sponsored by or endorsed by any of these companies.

## Data sources

| Source | Used for | Terms |
|---|---|---|
| [National Vulnerability Database (NVD), NIST](https://nvd.nist.gov/) | CVE details and CVSS scores for application vulnerabilities | This product uses the NVD API but is not endorsed or certified by the NVD. [NVD terms of use](https://nvd.nist.gov/developers/terms-of-use) |
| [Known Exploited Vulnerabilities (KEV) catalog, CISA](https://www.cisa.gov/known-exploited-vulnerabilities-catalog) | Flagging CVEs known to be exploited | Distributed under CC0 1.0. Use does not imply endorsement by CISA or DHS. [KEV license](https://www.cisa.gov/sites/default/files/licenses/kev/license.txt) |
| [Exploit Prediction Scoring System (EPSS), FIRST](https://www.first.org/epss/) | Likelihood that a CVE is exploited in the next 30 days | EPSS scores courtesy of FIRST.org and the EPSS Special Interest Group. [FIRST services terms of use](https://www.first.org/about/policies/terms) |
| [CVE Program](https://www.cve.org/) | CVE identifiers | [CVE terms of use](https://www.cve.org/Legal/TermsOfUse) |
| [Microsoft Security Response Center (MSRC) Security Update Guide](https://msrc.microsoft.com/update-guide) | Monthly Windows security updates and the CVEs they fix | [Microsoft terms of use](https://www.microsoft.com/en-us/legal/terms-of-use) |
| [CMHealthCheck](https://github.com/Skatterbrainz/CMHealthCheck) (Assets/windows_update_errorcodes.csv) | Descriptions of Windows Update error codes on the software update install failures page | © 2017-2024 Skatterbrainz. [MIT](https://github.com/Skatterbrainz/CMHealthCheck/blob/master/LICENSE) |
| [Windows release health, Microsoft Learn](https://learn.microsoft.com/en-us/windows/release-health/release-information) | Windows and Windows Server versions, builds and end-of-servicing dates | [Microsoft Learn terms of use](https://learn.microsoft.com/en-us/legal/termsofuse) |

## Open-source software

Steep is built on the following libraries. Each is used under its own license; the link goes to its source and license text.

| Library | Copyright | License |
|---|---|---|
| [.NET, ASP.NET Core and Blazor](https://github.com/dotnet/aspnetcore) | © .NET Foundation and contributors | [MIT](https://github.com/dotnet/aspnetcore/blob/main/LICENSE.txt) |
| [.NET runtime libraries (System.Management, System.DirectoryServices.Protocols and others)](https://github.com/dotnet/runtime) | © .NET Foundation and contributors | [MIT](https://github.com/dotnet/runtime/blob/main/LICENSE.TXT) |
| [Entity Framework Core and Microsoft.Data.Sqlite](https://github.com/dotnet/efcore) | © .NET Foundation and contributors | [MIT](https://github.com/dotnet/efcore/blob/main/LICENSE.txt) |
| [Microsoft.Data.SqlClient and Microsoft.SqlServer.Server](https://github.com/dotnet/SqlClient) | © Microsoft Corporation | [MIT](https://github.com/dotnet/SqlClient/blob/main/LICENSE) |
| [Microsoft.Data.SqlClient.SNI](https://www.nuget.org/packages/Microsoft.Data.SqlClient.SNI.runtime) | © Microsoft Corporation | [Microsoft Software License Terms (redistributable)](https://www.nuget.org/packages/Microsoft.Data.SqlClient.SNI.runtime/7.1.0/License) |
| [Microsoft IdentityModel extensions](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet) | © Microsoft Corporation | [MIT](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet/blob/dev/LICENSE.txt) |
| [Open XML SDK](https://github.com/dotnet/Open-XML-SDK) | © Microsoft Corporation | [MIT](https://github.com/dotnet/Open-XML-SDK/blob/main/LICENSE) |
| [ClosedXML](https://github.com/ClosedXML/ClosedXML) | © ClosedXML contributors | [MIT](https://github.com/ClosedXML/ClosedXML/blob/develop/LICENSE) |
| [ClosedXML.Parser](https://github.com/ClosedXML/ClosedXML.Parser) | © ClosedXML contributors | [MIT](https://github.com/ClosedXML/ClosedXML.Parser/blob/develop/LICENSE.txt) |
| [ExcelNumberFormat](https://github.com/andersnm/ExcelNumberFormat) | © ExcelNumberFormat contributors | [MIT](https://github.com/andersnm/ExcelNumberFormat/blob/master/LICENSE) |
| [RBush](https://github.com/viceroypenguin/RBush) | © Turning Code, LLC and others | [MIT](https://github.com/viceroypenguin/RBush/blob/master/LICENSE) |
| [SixLabors.Fonts 1.0](https://github.com/SixLabors/Fonts) | © Six Labors | [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0) |
| [AngleSharp](https://github.com/AngleSharp/AngleSharp) | © AngleSharp | [MIT](https://github.com/AngleSharp/AngleSharp/blob/devel/LICENSE) |
| [SQLitePCLRaw](https://github.com/ericsink/SQLitePCL.raw) | © SourceGear, LLC | [Apache 2.0](https://github.com/ericsink/SQLitePCL.raw/blob/master/LICENSE.TXT) |
| [SQLite](https://www.sqlite.org/) | Public domain | [Public domain](https://www.sqlite.org/copyright.html) |
| [Zensical (builds the offline help)](https://github.com/zensical/zensical) | © Zensical and contributors | [MIT](https://github.com/zensical/zensical/blob/master/LICENSE.md) |
| [Zensical UI (the offline help's theme, styles and scripts)](https://github.com/zensical/ui) | © Zensical and contributors | [MIT](https://github.com/zensical/ui/blob/master/LICENSE) |
| [clipboard.js (in the offline help)](https://github.com/zenorocha/clipboard.js) | © Zeno Rocha | [MIT](https://github.com/zenorocha/clipboard.js/blob/master/LICENSE) |
| [escape-html (in the offline help)](https://github.com/component/escape-html) | © TJ Holowaychuk, Andreas Lubbe, Tiancheng "Timothy" Gu | [MIT](https://github.com/component/escape-html/blob/master/LICENSE) |
| [focus-visible (in the offline help)](https://github.com/WICG/focus-visible) | © WICG | [W3C Software and Document License](https://github.com/WICG/focus-visible/blob/main/LICENSE.md) |
| [RxJS (in the offline help)](https://github.com/ReactiveX/rxjs) | © Google, Inc., Netflix, Inc., Microsoft Corp. and contributors | [Apache 2.0](https://github.com/ReactiveX/rxjs/blob/master/LICENSE.txt) |
| [Lucide icons (in the offline help)](https://github.com/lucide-icons/lucide) | © Lucide Icons and Contributors | [ISC](https://github.com/lucide-icons/lucide/blob/main/LICENSE) |
| [Font Awesome Free icons (in the offline help)](https://fontawesome.com/) | © Fonticons, Inc. | [CC BY 4.0](https://fontawesome.com/license/free) |
| [.NET runtime: third-party components](https://github.com/dotnet/runtime) | Various (see the notices) | [Third-party notices](https://github.com/dotnet/runtime/blob/main/THIRD-PARTY-NOTICES.TXT) |
| [ASP.NET Core: third-party components](https://github.com/dotnet/aspnetcore) | Various (see the notices) | [Third-party notices](https://github.com/dotnet/aspnetcore/blob/main/THIRD-PARTY-NOTICES.txt) |

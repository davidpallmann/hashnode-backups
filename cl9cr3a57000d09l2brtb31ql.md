# .NET Core 3.1 is nearing end-of-support

.NET Core 3.1 will reach end-of-life on December 13, 2022 - less than 60 days away from the date of this post. If you're still running production applications on .NET Core 3.1, you need to get moving.

## .NET Versions and Support

If you're not familiar with the .NET release cadence, here's some background. The original .NET Framework is Windows only. In 2014, .NET Core was announced, which made .NET cross-platform and open source. That meant you could run your .NET solutions on Linux and macOS as well as Windows, opening up a great many options. You can track .NET releases and support lifetimes at Microsoft's [.NET and .NET Core Support Policy](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core) page. When support ends, patches and security patches are no longer issued, so it's risky to remain on a non-supported version.

.NET Core 3.1 (December 2019-2022) was a very successful release that saw a high adoption rate. I think of it as when .NET Core entered the mainstream. With support ending, it's time to move to a later version. Now things get slightly confusing, because there was no .NET Core 4 (to avoid confusion with .NET Framework 4). Microsoft dropped the ".NET Core" naming convention and now calls modern .NET simply ".NET". Microsoft also adopted a release cadence that alternates between Long Term Support (3 years of support) versions and non-LTS (18 months of support) versions. The next version after .NET Core 3.1 is .NET 5 (November 2020-May 2022), but support has already ended for that non-LTS release. .NET 6 (November 2021-November 2024) is the logical release to move to.

| Version | Type | Release Date | End of Support | Status |
| -------- | ----- | ------ | ------ | ----- |
| .NET 6 | LTS | 2021-11-08 | 2024-11-12 | Supported |
| .NET 5 | Non-LTS | 2020-11-10 | 2022-05-10 | No longer supported |
| .NET Core 3.1 | LTS | 2019-12-03 | 2022-12-13 | Approaching end of support |

What can you expect in terms of code changes when moving from .NET Core 3.1 to .NET 6? There are a few changes, but they aren't earth-shattering. See [Migrate from ASP.NET Core 3.1 to 6.0](https://learn.microsoft.com/en-us/aspnet/core/migration/31-to-60?view=aspnetcore-6.0&tabs=visual-studio) for guidance. There are some nice new features in .NET 6, including minimal code, but you aren't required to use those features so none of that is a complication for upgrading.

## .NET 6 on AWS

If you run .NET workloads on AWS, moving to .NET 6 is straightforward. .NET 6 is supported on many AWS services. Refer to the [.NET 6 support on AWS guide](https://github.com/aws-samples/aws-net-guides/tree/master/RuntimeSupport/dotnet6) for details.

Many developers like to learn from code examples and tutorials. You can find lots of .NET 6 on AWS content, from AWS and community members, on the [.NET on AWS website community page](https://aws.amazon.com/developer/language/net/net-community/). 

| Channel | Content Type | Source |
|  ---- | ---- | ---- |
| [Basement Programmer](https://www.basementprogrammer.com/) | Blog, YouTube, Podcasts | Tom Moore, AWS Developer Advocate, Boston |
| [No Dogma](http://nodogmablog.bryanhogan.net/tag/aws/) | Blog, Podcast | Bryan Hogan, AWS Developer Advocate, Boston |
| [Code with Mukesh](https://codewithmukesh.com/blog/category/aws/) | Blog | Community member Mukesh Murugan |
| [Coding with Isaac](https://www.youtube.com/user/levini) | Podcast, YouTube | Isaac Levin, AWS Developer Advocate, Seattle |
| [François Bouteruche's blog](https://fbouteruche.medium.com/) | Blog | François Bouteruche, AWS Developer Advocate, Paris ]
| [Hello, Cloud](https://davidpallmann.hashnode.dev/series/hello-cloud) | Blog | David Pallmann, Manager Developer Advocacy, Texas |
| [Nick Chapsas Channel YouTube](https://www.youtube.com/playlist?list=PLUOequmGnXxOjsai24V-Ig0ZyEN_i9POx) | YouTube | Nick Chapsas, AWS Community Builder & Microsoft MVP |
| [Rahul Nath](https://www.rahulpnath.com) | Blog, YouTube | Rahul Nath, AWS Community Builder & Microsoft MVP, Australia | 
| [Wes Doyle YouTube Channel](https://www.youtube.com/playlist?list=PL3_YUnRN3UhgFuTi043IZlZkO2tD5gbqH) | YouTube | Wes Doyle, AWS Community Builder, Wisconsin |

.NET 6 is a good release, you can move to it with confidence. The time to let go of .NET Core 3.1 is now. There's no need to panic, but the time to upgrade is now.
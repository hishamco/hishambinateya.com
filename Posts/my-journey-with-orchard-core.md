---
Id: 65
Posted Date: 24/9/2019
Tags: OrchardCore
Slug: my-journey-with-orchard-core
---
# My Journey with Orchard Core

It has been a while since I wrote my latest blog post, because my left ankle was broken and did a surgey since then, the thing that let me stay on the bed for around half a year.

Afterward I did some contribution in OSS inluding [Orchard Core](https://https://github.com/OrchardCMS/OrchardCore) which I spent a lot of time on it with the community.

### Orchard Core RC Release

At the beginning of this blog post I'm glad to announce that the Orchard Core team will ship the RC release today - align with shipping .NET Core 3.0 release - after three beta releases.

The current release includes many features such as:

- Content Localization
- RTL admin theme
- Resources CDN
- Media CDN support
- GitHub authentication
- Facebook applications
- .NET Core 3.0
- Localization NuGet packages
- Azure Media resizing
- SQL fields indexing
- Full Text aspect

### Orchard Core & .NET Core 3.0

The **BIG** thing in this release that Orchard Core is now targeting `netcoreapp3.0` after shipping `.NET Core 3.0` today at `.NET Conf 2019`.

What that mean? This mean a lot in terms of performance improvement, feature wise such as new routing API and JSON APIs and much more ..

For more detailed information you can read the [Announcing .NET Core 3.0](https://devblogs.microsoft.com/dotnet/announcing-net-core-3-0/).

### Me & Localization ❤️ Orchard Core RC

I like working in localization since the `DNX` days where `Microsoft.Extensions.Localization` was introduced at the time, and I continue to do localization stuff with many open source projects, so what's new in localization with Orchard Core RC?

**1- Content Localization**

This release introducing a new `OrchardCore.Modules.ContentLocalization` module which allow the content author to create multi-lingual content items after enabling the module by the site admin.

This is simply happen by adding a `LocalizationPart` to any content item in which we need to support localization for that.

Just a little tip, don't be confuse with `OrchardCore.Modules.Localization` which allow the site to enable the localization feature but not localize the content.

**2- RTL admin theme**

Finally in this relase the admin site fully support RTL for those language whose language direction is right to left such as Arabic - which is mother language -, Persian and Hebrew.

To support the RTL in the admin theme, you just simply need to enable `OrchardCore.Modules.Localization` which I mentioned earlier.

**3- Localization NuGet packages**

Orchard Core translations are using [Crowdin](https://crowdin.com/project/orchard-core)  quite often for translating the content for the supported cultures. Last few weeks the Orchard Core team introduced a new set of translations NuGet packages in the [Orchard Translations](https://github.com/OrchardCMS/OrchardCore.Translations) repository to allow the developers localize thier sites simply by adding NuGet package.

### Working Experience with Orchard Core Community

Orchard Core community is fablous!! I never ever work with such community in open source project before.

Orchard Core is a huge project - and I'm not aware about the entire project and every module - but the community folks are very active in the Orchard Core repository as well as in gitter, so whenever I have an a question I can file an issue or write it in gitter and I got the answer very quickly.

I'd like to say that I'm glad to work with such teacm & community before and after I join the Orchard Core dev team. There 're many cool guys in the team that I like to thank them for their assistant such as: [Sébastien Ros](https://github.com/sebastienros), [Jean-Thierry Kéchichian](https://github.com/jtkech), [Jasmin Savard](https://github.com/Skrypt), [Jean-Philippe Tissot](https://github.com/jptissot), [Dean Marcussen](https://github.com/deanmarcussen) .. etc.

Also there 're many good friends from the community the made me busy with RTL issues :)


### My Contributions

I was one of the old follower for OrchardCore project since it named `Brochard`, I'm not sure if I made a PR at that time, but after awhile, why I contribute on [YesSQL](https://github.com/sebastienros/yessql) & [Fluid](https://github.com/sebastienros/fluid) projects I saw some activity going on Orchard project at the time I decided to do some contribution.

If I recall almost my contribution are:

- Localization
- Support RTL Admin Theme
- Translations Packages
- Prettify the Admin User Interface

### Lessons to be learned

- Priority .. priority .. priority as Seb always said :) , focusing on the issues within milestones that planned by the PM is very important in any OSS instead of picking random once to fix
- it's not a matter of writing a fix .. it's a matter of how you write it
- Suggest .. share ideas with the team instead of doing it lonely, because we working on open
- Write a unit tests whenever it's possible

Finally I enchourge everyone who didn't heard or work with Orchard Core to have a try.

Happy Coding !!
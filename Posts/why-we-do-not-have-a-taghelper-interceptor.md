---
Id: 43
Posted Date: 7/8/2017
Tags: TagHelpers,TagHelper Interceptor 
Slug: why-we-do-not-have-a-taghelper-interceptor
---
# Why we do not have a TagHelper Interceptor?

Last few days I come across a situation that I need to change the rendering output of one of the built-in tag helpers, frankly I didn't find a suitable way to accomplish my goal, after that I was thinking why we don't have ability for the interceptions the tag helpers generation, with that I decide to blog about the tag helper interception today.

### The need for the TagHelper Interception

As I mentioned above assume you need to change the rendering output for one tag helper for whatever a reason, what you will do?!! the quick and easiest answer for almost of us is by subclass the `{XXX}TagHelper` that you need it's output. In that case you can change the tag helper content after the super class tag helper is generating the its content.

May be this will fit your needs especially if you are trying to attaching some new properties, but that's not the case. What if you need to alter the output before the `{XXX}TagHelper` is executed?!!

`TagHelperInterceptor` is come for a rescue, this is one of the needs of such interception, AFAIK I never saw any method to allow me as developer to get into that level **(PreRender)**, so I can do any stuff before tag helper is prerender. Even I knew there's a property called `Order` which may useful in some cases but don't be confused, because this is fit where multiple tag helpers target the same element.

### TagHelper Interceptor & Code Generation

Another cool thing about having `TagHelperInterceptor` is the compilation process, till now you aren't able to participate in the compilation process for the tag helper code generation, because you don't have access the underlying APIs via `TagHelper`, perhaps some of you will said why is that important?!!

If you we such access, we can do many things beyond imagination such as creating iterator variables in `RepeaterTagHelper` or `GridViewTagHelper`. Also you can have a look to [Damian Edward](https://twitter.com/DamianEdwards)'s comment on `ImageTagHelper` [here](https://github.com/aspnet/Mvc/issues/2249#issuecomment-100026138) where such feature is possible using the `TagHelperInterceptor`.

Finally I'd like to share the following code snippet which a sample prototype for the `TagHelperInterceptor` inspired from `ControlBuilderInterceptor` in ASP.NET WebForms.
```csharp
public abstract class TagHelperInterceptor
{
     public virtual void PreTagHelperInit(
          TagHelper tagHelper,
          Type type,
          string tagHelperName) {

    }

     public virtual void OnProcessGeneratedCode(
          TagHelper tagHelper,
          TagHelperChunck tagHelperChunk,
          TagHelperDescriptor descriptor,
          CSharpCodeWriter writer,
          CodeGeneratorContext codeGeneratorContext) {

    }
}
```
Happy Coding !!
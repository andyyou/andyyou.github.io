---
layout: post
title: 'Model Binding Validation 資料模型的驗證機制'
date: 2013-03-13 05:10
categories: .NET Database
---


# 資料模型的驗證機制


在物件導向的世界裡有很多種方式可以驗證資料模型(Model Class) 。大部分的情況下我們可以在 Property
的 setter 裡面作資料的確認。就是如下程式碼

{% highlight csharp %}
public class Car
{
	private int _id;
	public int Id
	{
		get{return _id;}
		set{
			// validate here
			_id = value;
		}
	}
}
{% endhighlight %}

主要的原因是後續使用這個物件的時候不會遇到不符合規則的資料導致物件出例外。
第二個理由是
<!-- more -->
當我們要控制這個屬性的時候比較單純。
任何資料要設定到物件的屬性上都是透過 setter 而且可以確保資料是正確符合規範的。
事實上在 MVVM 架構下透過 setter 來處理例外和商業邏輯仍然是最簡單的選擇。
但假如有一個 Car 類別包含 Color 屬性且資料是根據另一個 Type 類別提供的，
然後您就會發現每當您使用 ORM 直接載入物件的時，如果想設定 Color 之前必須先設定 Type 的資料，因為驗證規則寫在 Type 的 setter。

在 ASP.NET MVC 架構下建議您不要直接在 setter 驗證。 
而是使用 ASP.NET MVC 內建的 ModelState ，它可以透過在 Model 屬性宣告規則或實作 IValidatableObject 然後當資料從 Form 表單透過 Http Request 傳入的時
 Model Binding 機制會在繫結時幫您確認資料是否有正確。
例如下面 Action，

{% highlight csharp %}
public ActionResult Edit(Car car){ ...
{% endhighlight %}

ModelState 讓您可以在 Controller 的 Action 中判斷資料，在資料儲存之前可以判斷是否有錯誤，然後處理。

{% highlight csharp %}
[HttpPost]
public ActionResult Edit(Car car)
{
	if(ModelState.IsValid)
	{
		// Update code to be placed here
		return RedirectToAction("CarList");
	}else
	{
		return View("CarEdit", car);
	}
}
{% endhighlight %}

上面的程式碼描述了關於編輯 Car 物件，程式碼的第一行 ModelState.IsValid 屬性會回傳 true/false 代表本次的驗證是否通過。
如果如果模型繫結發生錯誤，或者有商業邏輯上的錯誤， ASP.NET MVC 會告知您或者也可以使用 IValidatable 這個介面
實作 Validate Method 在這個 Method 裡面您需要自己實作商業邏輯。
每當一個錯誤發生，你就要把錯誤訊息加入<code>錯誤資訊集合</code>然後回傳。
下面是一個範例

{% highlight csharp %}
public class Car: IValidatableObject
{
	public int Id{get;set;}
	public string Name{get;set;}
	public string Type{get;set;}
	public string Color{get;set;}
	
	public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
	{
		if(string.IsNullOrEmpty(Name))
		{
			yield return new ValidationResult("Name is mandatory", new[] {"Name"});
		}
		if(string.IsNullOrEmpty(Type))
		{
			yield return new ValidationResult("Type is mandatory", new[]{"Type"});
		}
		if(string.IsNullOrEmpyt(Color))
		{
			yield return new ValidationResult("Color is mandatory", new[] {"Color"});
		}
	}
}
{% endhighlight %}

上面我們看到有三種屬性的驗證規則如果錯誤會透過 yield 敘述式將資訊回傳到集合中。這個<code>錯誤資訊集合</code>是一個泛型，
資料型別是 ValidationResult 這個類別讓您可以加入錯誤訊息。

{% highlight csharp %}
public class Car: IValidatableObject
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Type { get; set; }
    public string Color { get; set; }
 
    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        if(string.IsNullOrEmpty(Name))
        {
            yield return new ValidationResult("Name is mandatory", new[] {"Name"});
        }
        if (string.IsNullOrEmpty(Type))
        {
            yield return new ValidationResult("Type is mandatory", new[] { "Type" });
        }
        if (string.IsNullOrEmpty(Color))
        {
            yield return new ValidationResult("*", new[] { "Color" });
            yield return new ValidationResult("Color is mandatory"});
        }
    }
}
{% endhighlight %}

又或者您也可以使用 Data Annotation 的方式定義驗證規則

{% highlight csharp %}
public class Car
{
    public int Id { get; set; }
	[Required]
    public string Name { get; set; }
	[Required]
    public string Type { get; set; }
	[Required]
    public string Color { get; set; }
}
{% endhighlight %}

現在我們已經可以在執行資料模型繫結順便幫 Model 類別驗證，然後我們就在 Controller 的 Action 中控制整個流程。
而且關於資料驗證的規則我們依然把它寫在 Model 裡面。

我們當然不希望任何人可以編輯一個不能用的資料。這時我們就可以在 Action 先做處理。如果該資料不存在或不能用了
我們就使用 ModelState.AddModelError 追加錯誤訊息。

{% highlight csharp %}
[HttpPost]
public ActionResult Edit(Car car)
{
    if(IsCarAvailable())
    {
        ModelState.AddModelError(string.Empty,"Car cannot be edited because not available anymore");
    }
 
    if(ModelState.IsValid)
    {
        //Update code to be placed here
 
        return RedirectToAction("CarList");
    }
    else
    {
        return View("CarEdit",car);
    }
}
{% endhighlight %}

檢索所有的錯誤訊息；

{% highlight csharp %}
var allErrors = ModelState.Values.SelectMany(e => e.Errors).Select(gh => gh.ErrorMessage);
{% endhighlight %}

另外如果在 ASP.NET MVC 中我們也可以透過 Action 和 View 搭配檢查或實驗錯誤。我們先建立一個 Action 如下

{% highlight csharp %}
public ActionResult EditCar(Car car)
{
    // var errors = ModelState.Values.SelectMany(e => e.Errors).Select(gh => gh.ErrorMessage);
    ViewBag.Validation = ModelState.IsValid;
    return View();
}
{% endhighlight %}

然後再對應的 View 使用

{% highlight csharp %}
@ViewBag.Validation
@foreach (ModelState modelState in ViewData.ModelState.Values) {
    foreach (ModelError error in modelState.Errors) {
        <li>@error.ErrorMessage</li>
    }
}
{% endhighlight %}

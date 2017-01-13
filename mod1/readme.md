# Module 1: Building Your First Xamarin.Forms App
Descargar el codigo start. 

Botón derecho en "Solution explorer" y "Restore NuGet Packages"

Hacemos deploy: "Welcome to Xamarin.Forms!"

Revisar las carpetas que están vacías. Xamarin nos permitirá compartir entre el 70 y el 90% de nuestro código pero la lógica de la interfaz de usuario tendrá que ir en los proyectos de cada plataforma. El código compartido irá en un Shared Project o una PCL. 

La clase 'Aplicación' es el punto de entrada de las apps Xamarin.Forms. Esta clase selecciona la main page y gestiona el ciclo de vida de la misma. En nuestra clase podemos ver nuestros "OnStart", "OnSleep" y "OnResume"

Es normal preguntarse porque tenemos proyectos por plataforma si todo el código es compartido. Esto es porque las aplicaciones Xamarin.Forms son nativas y mantenemos la posibilidad de bajar a una plataforma para acceder al 100% de las apis nativas. Esto es necesario, por ejemplo, para manejar rutas para guardar elementos, o interactuar con elementos propios del sistema operativo. 

Si entramso en el proyecto Android y abrimos 'MainActivity' veremos las líneas que inicializan Xamarin Forms y cargan una nueva app. 
``` csharp
Xamarin.Forms.Forms.Init(this, bundle);
LoadApplication(new App());
```

##### Añadir el base view model
```csharp
using System;
using System.ComponentModel;

namespace Spent
{
	public class BaseViewModel : INotifyPropertyChanged
	{
		public BaseViewModel()
		{
		}

		public event PropertyChangedEventHandler PropertyChanged;
	}
}
```

**C# 6 (Visual Studio 2015 or Xamarin Studio on Mac)**
```csharp
using System.Runtime.CompilerServices;

public void OnPropertyChanged([CallerMemberName] string name = null) =>
    PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
```

**C# 5 (Visual Studio 2012 or 2013)**
```csharp
public void OnPropertyChanged([CallerMemberName] string name = null)
{
    var changed = PropertyChanged;

            if (changed == null)
                return;

            changed.Invoke(this, new PropertyChangedEventArgs(name));
}
```
De este modo podemos llamar a 'OnPropertyChanged' en el momento que una propiedad se actualice.

También algo básico es no pillarnos las manos duplicando peticiones, por ejemplo, si un usuario solicita actualizar 4 veces seguidas, debemos hacer sólo una petición de datos. Así que creamos una propiedad IsBusy

```csharp
private bool isBusy;
public bool IsBusy
{
    get { return isBusy; }
    set
    {
        isBusy = value;
        OnPropertyChanged();
    }
}
```


##### 3. Add expenses model.
Nuestra aplicación es para controlar los gastos, así que nos hará falta una clase que tenga la información de los mismos, creamos el modelo Expense. 

```csharp

namespace Spent
{
	public class Expense
	{
		public string Company { get; set; }
		public string Description { get; set; }
		public string Amount { get; set; }
		public DateTime Date { get; set; }
		public string Receipt { get; set; }
	}
}
```

This will keep track of the company, description, purchase amount, purchase time, as well as a string that represents a path to our receipt in the phone's local storage.

##### 4. Add a expenses view model.
En Spent vamos a ver la lista de gastos en que el usuario ha incurrido. Creamos por tanto el view model de nuestra primera pantalla. En la carpeta viewmodels la clase ExpensesViewModel que hereda de BaseViewModel

Add new collection to the class for storing our expenses and initialize it in the constructor.

```csharp
using System;
using System.Collections.ObjectModel;

namespace Spent
{
	public class ExpensesViewModel : BaseViewModel
	{
		public ObservableCollection<Expense> Expenses { get; set; }

		public ExpensesViewModel()
		{
			Expenses = new ObservableCollection<Expense>();
		}
	}
}
```

Ahora que ya tenemos nuestro Viewmodel para los gastos vamos a crear un método para recuperar los mismos. 

```csharp
using System.Threading.Tasks;
using Xamarin.Forms;
...
async Task GetExpensesAsync()
{
    if (IsBusy)
        return;

    IsBusy = true;

	try
	{
        // Load up user's expenses here.
	}
	catch (Exception ex)
	{
		MessagingCenter.Send(this, "Error", ex.Message);
	}
	finally
	{
		IsBusy = false;
	}
}
```

Añadimos algunos datos en forma de mockup 
```csharp
Expenses.Clear();
Expenses.Add(new Expense { Company = "Walmart", Description = "Always low prices.", Amount = "$14.99", Date = DateTime.Now });
Expenses.Add(new Expense { Company = "Apple", Description = "New iPhone came out - irresistable.", Amount = "$999", Date = DateTime.Now.AddDays(-7) });
Expenses.Add(new Expense { Company = "Amazon", Description = "Case to protect my new iPhone.", Amount = "$50", Date = DateTime.Now.AddDays(-2) });
```

Realzamos la llamada a nuestra petición de gastos en el constructor para asegurarnos de que se rellene. 
Ya tenemos datos en nuestra app, pero si el usuario quiere refrescar sólo puede reiniciarla. Así que vamos a añadir un comando para que pueda refrescar. 

```csharp
public Command GetExpensesCommand { get; set; }
```

Let's initialize the command in our constructor.

```csharp
GetExpensesCommand = new Command(
    async () => await GetExpensesAsync());
```

De este modo tenemos nuestro viewmodel totalmente completo 

##### 4. Add expenses page.
Para la UI en Xamarin.Forms se usa o C# o  XAML. Para las dos vertientes hay pros y contras, en mi caso me gusta XAML porque me permite mantener la implementación del patrón MVVM y me separa la lógica de view y viewmodel. Visualmente también me permite ver el árbol del diseño que me cuesta más en C#. 

Creamos por tanto ExpensesPage como Forms Xaml Page 

```csharp
<?xml version="1.0" encoding="UTF-8"?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms" xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml" x:Class="Spent.ExpensesPage">
	<ContentPage.Content>
		<ListView>

		</ListView>
	</ContentPage.Content>
</ContentPage>
```

Configuramos nuestro listview para que utilice nuestra observablecollection que definimos en el viewmodel 

```csharp
<ListView ItemsSource="{Binding Expenses}">

</ListView>
```

Esto hace que la información de nuestra lista esté en nuestro listview, así él ya conoce la información, así que vamos a decirle que y cómo mostrarlo creando un ItemTemplate. 

```csharp
<ListView ItemsSource="{Binding Expenses}">
	<ListView.ItemTemplate>
		<DataTemplate>

		</DataTemplate>
	</ListView.ItemTemplate>
</ListView>
```

El siguiente código es una plantilla para definir una celda que ya viene con Xamarin Forms. En nuestro caso se amolda perfectamente y ya viene definida. 

```csharp
<ListView ItemsSource="{Binding Expenses}">
	<ListView.ItemTemplate>
		<DataTemplate>
			<TextCell Text="{Binding Company}" Detail="{Binding Amount}" />
		</DataTemplate>
	</ListView.ItemTemplate>
</ListView>
```

Tenemos ya un listview conectado con nuestros datos. Cada celda es un Expense individual. Así que podemos conectarle directamente cualquiera de sus propiedades. Pero, ¿cómo sabe nuestra View qué objetos conectar? No tiene un contexto en que buscar los objetos. Así que tenemos que establecerle un BindingContext para que la vista sepa a donde referenciarse. Vamos a `ExpensesPage.xaml.cs` y lo establecemos. 

```csharp
public ExpensesPage()
{
	InitializeComponent();

	BindingContext = new ExpensesViewModel();
}
``` 

Lo ideal es que la única lógica escrita en el codebehind de la página sea esta asociación del BindingContext, o incluso, ni esto. 

Por último tendremos que decirle a nuestra app que se inicie con nuestra ExpensesPage:
```csharp
MainPage = new ExpensesPage();
```

[MOSTRAMOS APP]

Con esto ya tenemos nuestros gastos mostrándose. Pero y nuestro usuario¿? No tiene cómo refrescar sus datos. En Xamarin.Forms el pull-to-refresh está implementado ya en los listview sólo con configurar algunas propiedades. 

```csharp
<ListView ItemsSource="{Binding Expenses}"
	IsPullToRefreshEnabled="true"
	IsRefreshing="{Binding IsBusy, Mode=OneWay}"
	RefreshCommand="{Binding GetExpensesCommand}">
	<ListView.ItemTemplate>
		<DataTemplate>
			<TextCell Text="{Binding Company}" Detail="{Binding Amount}" />
		</DataTemplate>
	</ListView.ItemTemplate>
</ListView>
```

* `IsPullToRefreshEnabled`: Define si se permite hacer el pull 
* `IsRefreshing`: Define si el listview está en proceso de actualizarse. Sólo queremos que nuestro VM actualice a nuestra view, de modo que OneWay 
* `RefreshCommand`: Define el comando que se utilizará para actualizar. 

[MOSTRAMOS APP]

##### 5. Add expense detail page.

Ahora mismo sólo tenemos apenas un par de datos para nuestros gastos, nuestro listview debería poder llevarnos a un detalle de un gasto cuando lo pulsamos. Para ello vamos a comenzar creando una ExpenseDetailPage y aprovechándonos de los layouts que Xamarin Forms nos provee cogemos un StackLayout. 

```csharp
<?xml version="1.0" encoding="UTF-8"?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms" xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml" x:Class="Spent.ExpenseDetailPage">
	<ContentPage.Content>
		<StackLayout Padding="20">

		</StackLayout>
	</ContentPage.Content>
</ContentPage>
```

Dentro vamos a usar diferentes label para mostrar nuestra información y un control Image para mostrar el detalle de un gasto. 

```csharp
<?xml version="1.0" encoding="UTF-8"?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms" xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml" x:Class="Spent.ExpenseDetailPage">
	<ContentPage.Content>
		<StackLayout Padding="20">
			<Label Text="Company" TextColor="#4d4d4d"/>
			<Label Text="{Binding Expense.Company}"/>
			<Label Text="Description" TextColor="#4d4d4d"/>
			<Label Text="{Binding Expense.Description}" />
			<Label Text="Date" TextColor="#4d4d4d"/>
			<Label Text="{Binding Expense.Date}" />
			<Label Text="Amount" TextColor="#4d4d4d"/>
			<Label Text="{Binding Expense.Amount}" />
			<Label Text="Receipt" TextColor="#4d4d4d"/>
			<Image Source="{Binding Expense.Receipt}"/>
		</StackLayout>
	</ContentPage.Content>
</ContentPage>
```

Al igual que nos pasaba en la lista de gastos, tras realizar el binding en nuestra vista, tenemos que indicarle donde va a tener que buscar nuestro objeto Expense para mostrarlo en pantalla. 



```csharp
public partial class ExpenseDetailPage : ContentPage
{
	public Expense Expense { get; set; }

	public ExpenseDetailPage(Expense expense)
	{
		InitializeComponent();

		Expense = expense;
		BindingContext = this;
	}
}
```

Ahora nos toca navegar de nuestra lista a nuestro detalle. Podemos hacerlo usando la propiedad SelectedItem de nuestro ListView 
Great! Now it's time to implement navigation from our `ListView` to our detail page. We can do this by using the `SelectedItem` property of the `ListView`. Update `ExpensesPage.xaml` to data bind the `ListView.SelectedItem` property to `SelectedExpenseItem`.

```csharp
<ListView ItemsSource="{Binding Expenses}"
	IsPullToRefreshEnabled="true"
	IsRefreshing="{Binding IsBusy, Mode=OneWay}"
	RefreshCommand="{Binding GetExpensesCommand}"
	SelectedItem="{Binding SelectedExpenseItem, Mode=TwoWay}">
	<ListView.ItemTemplate>
		<DataTemplate>
			<TextCell Text="{Binding Company}" Detail="{Binding Amount}" />
		</DataTemplate>
	</ListView.ItemTemplate>
</ListView>
```

When the `SelectedItem` changes, we can navigate to the detail view for the `SelectedItem`. Open up `ExpensesViewModel` and add a property named `SelectedExpenseItem`.

```csharp
Expense selectedExpenseItem;
public Expense SelectedExpenseItem
{
	get { return selectedExpenseItem; }
	set
	{
		selectedExpenseItem = value;
		OnPropertyChanged();

		if (selectedExpenseItem != null)
		{
			// TODO: Navigate to detail.
			SelectedExpenseItem = null;
		}
	}
}
```

In this code, we are updating the `SelectedExpenseItem`, and firing `OnPropertyChanged`. If the selected item is not null, we want to navigate to the detail page. Finally, we set the value to `null` to remove any cell highlighting that happens when a user taps the cell.

To handle navigation, we will need a reference to a `Page`. Let's add a class-level field typed `ExpensesPage`, add a parameter to the constructor of `ExpensesViewModel` to take in an `ExpensesPage`, and this equal to our new `ExpensesPage` field.

```csharp
ExpensesPage page;
public ExpensesViewModel(ExpensesPage expensesPage)
{
	page = expensesPage;
	...
}
```

Next, let's update our `SelectedExpenseItem` property to navigate when the value is changed.

```csharp
Expense selectedExpenseItem;
public Expense SelectedExpenseItem
{
	get { return selectedExpenseItem; }
	set
	{
		selectedExpenseItem = value;
		OnPropertyChanged();

		if (selectedExpenseItem != null)
		{
			ExpensesPage.Navigation.PushAsync(new ExpenseDetailPage(SelectedExpenseItem));
			SelectedExpenseItem = null;
		}
	}
}
```

We also need to update `Expenses.xaml.cs` to pass in `this` as a value to the `ExpensesViewModel` constructor.

```csharp
public partial class ExpensesPage : ContentPage
{
	public ExpensesPage()
	{
		InitializeComponent();

		BindingContext = new ExpensesViewModel(this);
	}
}
```

You may have caught that we are using a `Navigation` property of the page to push a new detail page onto the navigation stack. Right now, `ExpensesPage` is a `ContentPage`, which doesn't contain any ability to do navigation. To gain this ability, we must wrap our `ExpensesPage` in a [`NavigationPage`](https://developer.xamarin.com/api/type/Xamarin.Forms.NavigationPage/) to gain the ability to do push/pop and modal navigation. Jump over to `App.xaml.cs` and update the `MainPage` to the following.

```csharp
MainPage = new NavigationPage(new ExpensesPage());
```

By doing this, our `ExpensesPage.Navigation` property is now available for use! Additionally, by using a `NavigationPage`, we get a nice navigation bar at the top of all of our pages to alert users to what page they are on, and to navigate back to the top of the stack (with a "back" button). To have a title display in the navigation bar, we can update the `Title` property of each page to the appropriate value.

`**ExpensesPage**`
```csharp
<?xml version="1.0" encoding="UTF-8"?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms" xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml" x:Class="Spent.ExpensesPage"
		Title="Expenses">
...
</ContentPage>

```

`**ExpenseDetailPage**`
```csharp
<?xml version="1.0" encoding="UTF-8"?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms" xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml" x:Class="Spent.ExpenseDetailPage"
		Title="Expense Detail">
...
</ContentPage>
```

Now, run the application, click on an expense cell, and you will be navigated to a detail page with more information about our expense!

 ![](/modules/module-1/images/expenses-detail-view.png)

##### 6. Navigation with the Messaging Center.
Right now, we have a working master-detail navigation flow that shows a list of expenses, as well as detailed information about each expense. We can clean this up to be even better and reduce tight coupling between our views and view models.

Xamarin.Forms [`MessagingCenter`](https://developer.xamarin.com/guides/xamarin-forms/messaging-center/) enables view models and other components to communicate without having to know anything about each other besides a simple message contract. There are two main parts to the `MessagingCenter`:

1. **Subscribe**: Listen for messages with a certain signature and perform some action when they are received. Mulitple subscribers can be listening for the same message.
2. **Send**: Publish a message for listeners to act upon. If no listeners have subscribed then the message is ignored.

Instead of passing a `Page` around to handle navigation, what if we used the Xamarin.Forms `MessagingCenter` to handle this? Let's update our current app to use this approach.

Let's start by undoing some of the "harm" we have done! Open `ExpensesViewModel` and remove the parameter from the constructor, as well as the `ExpensesPage` field and all references to it in `SelectedExpenseItem`. Jump over to `ExpensesPage.xaml.cs` and update initialization of the `ExpensesViewModel` to have no parameters. 

Next, let's open back up `ExpensesViewModel` and send our first message! Let's send a message by using the following method signature `MessagingCenter.Send(TSender sender, string message, TArgs args)`. 

```csharp
Expense selectedExpenseItem;
public Expense SelectedExpenseItem
{
	get { return selectedExpenseItem; }
	set
	{
		selectedExpenseItem = value;
		OnPropertyChanged();

		if (selectedExpenseItem != null)
		{
			MessagingCenter.Send(this, "NavigateToDetail", SelectedExpenseItem);
			SelectedExpenseItem = null;
		}
	}
}
```

Note that we pass in `this` as the sender, "NavigateToDetail" as the message, and our selected `Expense` object as an argument. Sending a message is great, but if nobody is listening for messages of that signature, nothing will happen. Jump back over to `ExpensesPage.xaml.cs` and add two new methods: `SubscribeToMessages` and `UnsubscribeFromMessages`. Also, override the `OnAppearing` and `OnDisappearing` lifecycle methods of our `ContentPage` to call `SubscribeToMessages` and `UnsubscribeFromMessages`.

```csharp
protected override void OnAppearing()
{
	base.OnAppearing();

	SubscribeToMessages();
}

protected override void OnDisappearing()
{
	base.OnDisappearing();

	UnsubscribeFromMessages();
}

void SubscribeToMessages()
{

}

void UnsubscribeFromMessages()
{
		
}
```

It's important that we properly subscribe and unsubscribe to messages from the `MessagingCenter` to avoid null references and possible memory leaks. Let's subscribe to the message sent from our view model in `SubscribeToMessages`.

```csharp
void SubscribeToMessages()
{
	MessagingCenter.Subscribe<ExpensesViewModel, Expense>(this, "NavigateToDetail", async (obj, expense) =>
	{
		if (expense != null)
		{
			await Navigation.PushAsync(new ExpenseDetailPage(expense));
		}
	});

	MessagingCenter.Subscribe<ExpensesViewModel, string>(this, "Error", (obj, s) =>
	{
		DisplayAlert("Error", s, "OK");
	});
}
```

This subscribes us to messages with string "NavigateToDetail" coming from `ExpensesViewModel` with argument(s) `Expense`. If we receive a message, we first check that the argument isn't null, then navigate to our detail page. 

We can easily unsuscribe in `UnsubscribeFromMessages` as well.

```csharp
MessagingCenter.Unsubscribe<ExpensesViewModel, Expense>(this, "NavigateToDetail");
MessagingCenter.Unsubscribe<ExpensesViewModel, string>(this, "Error");
```

Run the app, and navigation should still be working as intended - only this time we are properly avoiding tight coupling between our view model and view.

#### Closing Remarks
In this module, you learned about the basics of building apps with Xamarin.Forms, including creating user interfaces in XAML, navigation, MVVM, data binding and commanding, as well as use of the `MessagingCenter`. In the next module, we'll take a look at polishing off our Xamarin.Forms app before connecting it to the cloud in Modules 3 and 4.

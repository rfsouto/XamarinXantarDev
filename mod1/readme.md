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

##### 4. Add a expenses view model.
En Spent vamos a ver la lista de gastos en que el usuario ha incurrido. Creamos por tanto el view model de nuestra primera pantalla. En la carpeta viewmodels la clase ExpensesViewModel que hereda de BaseViewModel

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

Ahora nos toca navegar de nuestra lista a nuestro detalle. Podemos hacerlo usando la propiedad SelectedItem de nuestro ListView en nuestro ExpensesPage.xaml para conectarlo a la propiedad SelectedExpenseItem. 

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

Cuando nuestro SelectedItem podemos navegar al detalle. Abrimos nuestro ExpensesViewModel y añadimos la propiedad SelectedExpenseItem

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

En este código estamos actualizando SelectedExpenseItem y lanzando nuestro OnPropertyChanged. Si el elemento no es nulo queremos entonces navegar al siguiente elemento. Al final ponemos el valor a null para eliminar cualquier marca futura en el propio listview. 

Para manejar la navegación necesitamos referenciar a una página. Añadimos el campo de tipo ExpensePage y lo nombramos cómo page, añadimos al mismo tiempo el parámetro al constructor ExpensesViewModel y lo igualamos a nuestro campo. 

```csharp
ExpensesPage page;
public ExpensesViewModel(ExpensesPage expensesPage)
{
	page = expensesPage;
	...
}
```

Ahora nos toca actualizar nuestra propiedad SelectedExpenseItem para que navegue cuando cambie el valor.

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

También tendremos que actualizar nuestra vista para indicarle con this, con qué página se tiene que inicializar el constructor de ExpensesViewModel. 

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

En el punto anterior, en la navegación, usamos la propiedad Navigation que no está habilitada puesto que ExpensesPage es una ContentPage y no una NavigationPage. Para que nuestra página gane esa habilidad debemos envolver nuestra ExpensesPage en una NavigationPage de la siguiente manera en nuestro App.xaml.cs : 

```csharp
MainPage = new NavigationPage(new ExpensesPage());
```

Ahora sí que nuestra propiedad está desponible para usar. Utilizando además NavigationPage obtenemso una barra de navegación en la parte superior que muestra el título de donde nos encontramos y permite volver al inicio del stack con un botón Atrás. 
Para añadir un título en la barra de navegación, basta con cambiar la propiedad Title en cada página por el valor adecuado. 

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

[MOSTRAMOS APP]

##### 6. Navigation utilizando Messaging Center.
Ahora mismo ya tenemos una navegación entre maestro y detalle. Pero aún podemos reducir la interrelación que hay entre views y viewmodels. 
Messaging Center permite a los view models y otros componentes comunicarse sin saber nada de los demás, con un simple mensaje. 

Existen dos partes principales en MessagingCenter;
1. **Subscribe**: Escucha mensajes con una firma y realiza una acción cuando estos se reciben. 
2. **Send**: Publica un mensaje a los escuchadores.

Así que vamos a actualizar nuestra navegación para usarlo. 

En ExpensesViewModel eliminamos el parámetro del constructor y nuestro campo, y todas las referencias a él en SelectedExpenseItem. Ahora vamos a la página ExpensesPage y cambiamos la inicialización de ExpensesViewModel sin parámetros. 

Volvemos al viewmodel y enviamos nuestro primer mensaje, vamos a enviar un mensaje usando el siguiente método 
`MessagingCenter.Send(TSender sender, string message, TArgs args)`. 

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

Enviar un mensaje está bien, pero el truco reside en que alguien nos escuche. Volvemos a nuestra página y vamos a añadir dos nuevos métodos:
SubscribeToMessages y UnsubscribeFromMessages. También haremos override de OnAppearing y OnDisappearing para llamar a nuestros nuevos métodos. 

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

Es importante suscribirse y desuscribirse de los mensajes del MessagingCenter para evitar referencias nulas y problemas de memoria. 
Nos suscribimos al mensaje enviado por nuestro viewmodel en SubscribeToMessages

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
Esto nos suscribe a los mensajes que sean NavigateToDetail desde ExpenseViewModel con un argumentno Expense. Al recibir el mensaje validamos que sea nulo y luego completamos la navegación. 


Igual de fácil es realizar la desuscripción. 

```csharp
MessagingCenter.Unsubscribe<ExpensesViewModel, Expense>(this, "NavigateToDetail");
MessagingCenter.Unsubscribe<ExpensesViewModel, string>(this, "Error");
```

[MOSTRAMOS APP]




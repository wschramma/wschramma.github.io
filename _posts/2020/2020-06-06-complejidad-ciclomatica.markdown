---
layout: post
author: wschramm
title:  "Complejidad cilomática Parte 1"
date:   2020-06-06 11:20:23
categories: [conceptos/patrones/buenas-practicas]
tags: [comlejidad-ciclomatica]
---

La complejidad ciclomática es una de las medidas básicas para conocer lo complejo que es un código. 
Básicamente es un indicador del número de caminos que puede ejecutar un segmento de código, es decir que en base a esta métrica podremos determinar si un segmento de nuestro código esta tomando demasiadas decisiones.

La medición de la complejidad se define a través de algunos elementos comunes de los lenguajes de programación que pueden hacer que se ejecuten una u otra parte del código.
Los bucles como el **for** o el **while** agregan complejidad porque hay distintas rutas de ejecución a través o alrededor de ellos.
Las instrucciones **if-else** anidadas o las que se encuentran dentro de bucles agregan aún más complejidad al código.

# Un ejemplo

Imaginemos que tenemos una aplicación de ventas. La capa de presentación maneja diferentes solicitudes como el inicio de sesión o la compra de un artículo.
El servicio de aplicación controla estas solicitudes, veamos solo la implementación del método de compra.

```c#

public class ApplicationServices: IApplicationServices{
  private readonly IDomainService _domain;
  
  public ApplicationServices(IDomainService domain){
    _domain = domain;
  }

  public ReceiptDto LoggedInUserPurchase(string itemName){
    if (IsDownForMaintenance())
      return null;
    return _domain.Purchase(Session.LoggedInUsername, itemName);
  }

  private bool IsDownForMaintenance(){
    return File.Exists("maintenance.lock");
  }
}
```

El método **LoggedInUserPurchase** realiza la operación de compra de un elemento a nombre del usuario que inicio sesión. Devolviendo un **DTO** (objeto de transferencia de datos), el cual tiene la información de la compra.

Cómo se ve este método es bastante simple, sin embargo la implementación se bifurca porque la implementación admite un modo de mantenimiento. Si existe un archivo específico llamado **maintenance.lock**, el servicio simplemente cerrará la operación y devolverá null.

De igual forma la compra puede fallaren la capa de dominio, aquí esta la implementación de ejemplo de la clase de servicio de dominio:

```c#

  public class DomainServices: IDomainServices{
    private readonly IUserRepository    _userRepository;
    private readonly IProductRepository _productRepository;
    private readonly IAccountRepository _accountRepository;
    
    public ReceiptDto Purchase(string userName, string itemName){
      var user = _userRepository.Find(userName);
      
      if (user is null)
        return null;
        
      var acount = _accountRepository.FindByUser(user);
      return this.Purchase(user, account, itemName);
    }
  
    private ReceiptDto Purchase(User user, Account account, string itemName){
      var item = _productRepository.Find(itemName);
      
      if (item is null)
        return null;
        
      var receipt     = user.Purchase(item);
      var transaction = account.Withdraw(receipt.Price);
      
      if (transaction is null)
        return null;
        
      return receipt;
    }
  }

```

La primera tarea que realiza el servicio de dominio es buscar al usuario por su nombre. Si el usuario no existe, devuelve un null. En caso contrario, la operación continúa, pero puede producirse otro error sí la cuenta no existe. La tercera forma de fallar es que la transacción monetaria falle (por ejemplo como resultado de un saldo bajo en la cuenta del usuario). Por último, si se pasaron todas las validaciones, el método devolverá un DTO.

La consecuencia directa de devolver null de los métodos es que quien llama a este método deberá tratar con el resultado nulo por separado de los demás resultados no nulos. Está es una posible implementación del controlador :

```c#

  public class HomeController : Controller{
    private readonly IApplicationServices _application;
    
    public HomeController(IApplicationServices application){
      _appliation = application;
    }
    
    public ActionResult Purchase(string itemName){
      var receipt = _application.LoggedInUserPuerchase(itemName);
      
      return SelectView(receipt);
    }
    
    private ActionResult SelectView(ReceiptDto receipt){
      if (receipt is null)
        return new FailedPurchaseView();
      else
        return new SuccessfulPurchaseView(receipt);
    }
  }

```

El controlador es responsable de seleccionar la vista adecuada que se mostrara. El problema es que trata el resultado nulo por separado de los resultados no nulos.

# ¿Dónde esta el problema?

Existen dos problemas que son inherentes al uso de valores **NULL** en una aplicación.

El primer problema es que null no lleva ninguna información.¿Cuál fue la cauda de que la solicitud de compra fallara?,¿No habían fondos suficientes en la cuenta, o quizás fue un problema con la autenticación del usuario?. La capa de presentación no puede decirnos eso, solo puede decir que algo salió mal.

El segundo problema está en el código qué consume referencias nulas. Toda la lógica debe protegerse de los valores **null**. Cada instrucción que funciona en un objeto potencialmente nulo se convierte en una instrucción **if-else**. Esto puede aumentar significativamente la complejidad ciclomática del código, especialmente en aplicaciones más complejas.

# ¿Cómo deshacerse de los resultados nulos?

El enfoque más obvio para los resultados nulos es reemplazarlos con otros objetos. El patrón **null object** nos ayuda con eso.

La idea es construir un objeto ficticio que permanezca en lugar de un objeto normal en los casos en que no se pueda construir el objeto normal. También podríamos decir que **null object** es un reemplazo de la referencia nula.

La diferencia entre la implementación de **null object**  y la referencia nula es que **null object** sigue siendo un objeto y puede llevar cierta información. La información más obvia que cada objeto lleva es su tipo. Pero hay otras piezas de información proporcionadas por implementaciones predeterminadas en sus miembros.

# Ejemplo de implementación de objeto nulo

Supongamos que queremos reemplazar nuestro **PurchaseDTO** con un **null object** en caso que no se pueda realizar la compra. Este objeto nulo nos permitiría contar una historia como "no hay recibo, la compra no se realizó".

Lo primero que debemos realizar es que **PurchaseDTO** y **Null Purchase Object** deben ser reemplazables. En otras palabras, deben compartir un tipo base común.

Por lo tanto debemos comenzar la implementación definiendo primero un tipo base:

```c#
  public interface IReceiptViewModel{
  }
```

Ahora podemos proporcionar dos implementaciones de este ViewModel. La primera implementación ya existe (es el DTO de recepción). El único cambio es dejar que implemente la interfaz:

```c#
  public class ReceiptDto: IReceiptViewModel{
    
    public string   UserName  { get; private set; }
    public string   ItemName  { get; private set; }
    public decimal  Price     { get; private set; }
    
    public ReceiptDto(string userName, string itemName, decimal price){
      UserName  = username ?? "anonymous buyer";
      ItemName  = itemName;
      Price     = price;
    }
  }
```

La otra implementación será el objeto nulo:

```c#
  public class ReceiptNullObject: IReceiptViewModel{
    private static IReceiptViewModel instance;
    
    private ReceiptNullObject() { }

    public static IReceiptViewModel Instance
    {
        get
        {
            if (instance is null)
                instance = new ReceiptNullObject();
            return instance;
        }
    }
  }

```

La clase **ReceiptNullObject** es una clase vacía. Además, implementa el patrón **Singleton** que permitirá que siempre devuelva la misma instancia.

# Uso del objeto null

Para seguir con la implementación del patrón **null object** deberemos reemplazar todas las referencias nulas con la instancia de **null object**. En la aplicación de ejemplo, el servicio de aplicación solía devolver la recepción nula en caso de que la aplicación este abajo por mantención, corregiremos eso:

```c#
  public class ApplicationServices: IApplicationServices{
    ...
    public IReceiptViewModel LoggedInUserPurchase(string itemName){
        if (IsDownForMaintenance())
            return ReceiptNullObject.Instance;
        return _domain.Purchase(Session.LoggedInUserName, itemName);
    }
    ...
  }
```

El siguiente paso es repetir el proceso en los servicios de dominio:

```c#
  public class DomainServices: IDomainServices{

    private readonly IUserRepository    _userRepository;
    private readonly IProductRepository _productRepository;
    private readonly IAccountRepository _accountRepository;

    public ReceiptDto Purchase(string userName, string itemName){
        var user = _userRepository.Find(userName);

        if (user is null)
            return ReceiptNullObject.Instance;

        var account = _accountRepository.FindByUser(user);

        return this.Purchase(user, account, itemName);
    }

    private ReceiptDto Purchase(User user, Account account, string itemName){
        var item = _productRepository.Find(itemName);

        if (item is null)
            return ReceiptNullObject.Instance;

        var receipt     = user.Purchase(item);
        var transaction = account.Withdraw(receipt.Price);

        if (transaction is null)
            return ReceiptNullObject.Instance;

        return receipt;
    }

}

```

Con estos cambios no existen rutas que devuelvan null a la capa de presentación. Esto nos permitirá refactorizar el controlador para que no pregunte qué tipo de resultado recibió.

El controlador pude crear un map que determine qué vista utilizara en cada caso:

```c#
  public class HomeController: Controller{
    private readonly IApplicationServices                           _application;
    private Dictionary<Type, Func<IReceiptViewModel, ActionResult>> _receiptTypeToActionResult;

    public HomeController(IApplicationServices application){
        _application = application;

        _receiptTypeToActionResult = new Dictionary<Type, Func<IReceiptViewModel, ActionResult>>
                                    {
                                      {typeof(ReceiptDto), model => new SuccessfulPurchaseView(model as ReceiptDto)},
                                      {typeof(ReceiptNullObject), model => new FailedPurchaseView()}
                                    };
    }
    ...
}
```

Con esta modificación todo el problema de asignar resultados a vistas se convierte en una tarea de configuración. De ser necesario este fragmento de código se pude hacer aún más genérico y mover a una clase especializada que configure de forma transparente el controlador.

Ahora el controlador puede fácilmente seleccionar la vista para cualquiera de los DTO admitidos que se puedan representar en la respuesta:

```c#
  public class HomeController : Controller{
    ...
    public ActionResult Purchase(string itemName){
        var receipt = _application.LoggedInUserPurchase(itemName);
        return SelectView(receipt);
    }

    private ActionResult SelectView(IReceiptViewModel receipt){
        var modelType = receipt.GetType();
        var mapper    = _receiptTypeToActionResult[modelType];
        
        return mapper(receipt);
    }
}
```

Ahora no hay más **if-else** en el método **SelectView**. Este método ahora puede servir la vista basada en DTO que llego desde la capa de aplicación. La complejidad ciclomática de este método es ahora de 1 y seguirá siendo 1 después de agregar nuevas vistas y nuevos DTO a la aplicación.

# Conclusión

Una de las herramientas más sencillas que podemos usar para reducir la complejidad ciclomática es evitar las instrucciones **if-else** basadas en la comprobación de si alguna referencia es nula o no.

El patrón de diseño **null object** es muy útil para este tipo de situaciones, ayudando a mejorar la compresión y mantenibilidad del código.



# BadCodeBinas
Aquí tienes una descripción de un caso de uso para un sistema MVC en .NET 8, acompañado de un conjunto de problemas en código "malo" que deben resolverse para cumplir con los objetivos de desarrollo. Este caso está diseñado para que los estudiantes puedan identificar y aplicar patrones de diseño de la GoF y técnicas de refactorización.

---

## Caso: Sistema de Gestión de Inventario para Tiendas de Electrónica

El sistema tiene que permitir a los administradores de tiendas agregar, actualizar y eliminar productos, así como visualizar detalles sobre el inventario. Los usuarios finales pueden ver información básica de los productos disponibles. El sistema debe gestionarse a través de una estructura MVC para mantener la separación de lógica de negocio, presentación y acceso a datos.

### Objetivos del Caso y Problemas Identificados

El sistema presenta problemas en las áreas de estructura de código, modularidad y uso de patrones de diseño, lo que lo hace difícil de mantener y escalar. En cada punto, se sugiere una estrategia de refactorización y el patrón GoF más adecuado.

| **Problema**                              | **Descripción**                                                                                                                                                                                                 | **Posible Solución / Patrón**                                           |
|-------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|
| 1. **Métodos Largos**                     | El controlador de inventario tiene métodos que sobrepasan 50 líneas de código, acumulando varias responsabilidades.                                                                                             | **Refactorización con "Extract Method"** para dividir lógica compleja【43†source】. Utilizar el patrón **Command** para separar acciones específicas【34†source】. |
| 2. **Clases God**                         | La clase "ProductController" gestiona operaciones de lógica de negocio y validación de datos directamente en el controlador.                                                                                   | Aplicar **SRP** y crear servicios específicos para la lógica de negocio y validación utilizando el patrón **Facade**【36†source】. |
| 3. **Dependencias Fuertemente Acopladas** | El controlador depende directamente de varias clases, como ProductRepository y LoggingService, dificultando pruebas y mantenibilidad.                                                                           | Implementar **Inversión de Dependencia** y **Inyección de Dependencias** con contenedores IoC【39†source】. Utilizar el patrón **Strategy** para cambiar la lógica sin modificar el controlador【38†source】. |
| 4. **Código Repetido**                    | Las validaciones de datos se repiten en múltiples controladores, aumentando el riesgo de inconsistencias.                                                                                                      | Extraer lógica de validación en un servicio y utilizar **Chain of Responsibility** para encadenar reglas de validación【43†source】. |
| 5. **Mal Manejo de Errores**              | Las excepciones se capturan y manejan en cada método del controlador con bloques `try-catch` repetitivos, generando redundancia.                                                                               | Implementar un **Middleware de Excepciones** o utilizar el patrón **Observer** para manejar errores globalmente【37†source】. |
| 6. **Persistencia de Datos en Controlador** | El controlador guarda y carga datos directamente usando `Entity Framework`, sin separación clara entre el controlador y el modelo de datos.                                                                    | Aplicar **Repository Pattern** y **Unit of Work** para la gestión de datos【39†source】. |
| 7. **Uso de Constantes en Código**        | Varias configuraciones se definen directamente como constantes en el código, en lugar de utilizar un archivo de configuración.                                                                                 | Mover configuraciones a `appsettings.json` o implementar el patrón **Singleton** para la configuración global【42†source】. |
| 8. **No Uso de Interfaces**               | Las clases concretas se utilizan en lugar de interfaces, lo cual limita la flexibilidad para realizar pruebas y cambios.                                                                                       | Implementar **Programación a una Interfaz** en lugar de a una implementación concreta, para aumentar la flexibilidad【44†source】. |
| 9. **Carga Directa de Datos en Vista**    | La vista carga datos sin pasar por el controlador, violando el modelo MVC y limitando la capacidad de prueba.                                                                                                  | Usar el patrón **DTO (Data Transfer Object)** para transferir datos entre la capa de negocio y la vista【36†source】. |
| 10. **Lógica de Negocio en la Vista**     | La vista contiene cálculos y lógica de negocio, haciendo difícil su reutilización y mantenibilidad.                                                                                                           | Mover la lógica de negocio a la capa de modelo o servicios, y aplicar **MVVM (Model-View-ViewModel)** si es necesario【41†source】. |

### Consideraciones de Refactorización y Aplicación de Patrones

1. **Separación de Responsabilidades (SRP)**: Cada clase debe tener una sola responsabilidad【39†source】.
2. **Uso de Facade y Factory**: Facilita la creación de objetos y la agrupación de operaciones complejas en métodos simplificados【34†source】.
3. **Modularización**: Utilizar patrones de diseño estructurales como **Facade** para mantener las interfaces simples y **Adapter** para compatibilidad con sistemas heredados【40†source】.

### Ejemplo de Código "Malo"

```csharp
public class ProductController
{
    private ProductRepository productRepo = new ProductRepository();
    
    public void AddProduct(string name, string description, decimal price)
    {
        if (string.IsNullOrWhiteSpace(name) || price <= 0)
        {
            Console.WriteLine("Invalid data");
            return;
        }
        
        var product = new Product { Name = name, Description = description, Price = price };
        productRepo.Add(product);
        Console.WriteLine("Product added successfully");
    }
    
    public void ShowAllProducts()
    {
        var products = productRepo.GetAll();
        foreach (var product in products)
        {
            Console.WriteLine($"{product.Name}: {product.Description} - ${product.Price}");
        }
    }
}
```

---

### Recomendaciones de Recursos de Estudio

Para comprender mejor estos patrones y técnicas de refactorización, consulta:

- **"Dive Into Refactoring"** de Alexander Shvets【43†source】.
- **"Patrones de Diseño"**【42†source】 para repasar los patrones de la GoF y su aplicación práctica en C#.
- **"Head First Software Architecture"** para una introducción amigable al diseño arquitectónico en sistemas complejos【41†source】. 

Este ejercicio permitirá a los estudiantes aplicar patrones de diseño y principios SOLID para mejorar el diseño y arquitectura del sistema, logrando un código más flexible, modular y mantenible.

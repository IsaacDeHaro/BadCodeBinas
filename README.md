# BadCodeBinas

Aquí tienes un caso de código en una arquitectura MVC para .NET 8 con problemas comunes. Los objetivos de mejora están detallados para que podamos resolverlos uno a uno con un esfuerzo intermedio. A continuación, te doy una tabla con los objetivos y, después, el código que contiene estos errores.

### Tabla de Objetivos de Mejora

| **#** | **Objetivo de Mejora**                                                                                 | **Descripción**                                                                                                      |
|-------|--------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------|
| 1     | Separar lógica de negocios de los controladores                                                        | La lógica de negocios está mezclada con el controlador, dificultando su mantenimiento y pruebas unitarias.          |
| 2     | Implementar Dependency Injection (DI)                                                                   | Las dependencias se crean directamente en el controlador, generando acoplamiento y dificultad para hacer testing.    |
| 3     | Optimizar las consultas a la base de datos                                                              | Las consultas se ejecutan en bucles innecesarios, afectando el rendimiento.                                         |
| 4     | Manejar excepciones de forma adecuada                                                                   | El código no maneja excepciones, lo que puede provocar caídas de la aplicación sin mensajes claros.                  |
| 5     | Validar entradas de usuario en el modelo y el controlador                                               | Actualmente, no hay validación de datos de entrada, lo que deja vulnerabilidades en la aplicación.                   |
| 6     | Utilizar Data Transfer Objects (DTOs) en lugar de enviar modelos completos a la vista                   | El modelo completo se pasa directamente a la vista, exponiendo datos innecesarios.                                  |
| 7     | Implementar mapeo automático (AutoMapper) para la conversión de datos                                   | La conversión de datos entre modelo y DTO se hace manualmente, ocupando código repetitivo.                          |
| 8     | Mejorar el manejo del ciclo de vida de los objetos en el controlador                                   | Los objetos no se liberan adecuadamente, generando posibles fugas de memoria.                                       |
| 9     | Aplicar principios SOLID                                                                               | El código no respeta principios como el de Responsabilidad Única (SRP) y Abierto/Cerrado (OCP), afectando su diseño. |
| 10    | Añadir comentarios y mejorar la documentación del código                                               | El código carece de comentarios explicativos, dificultando su entendimiento.                                         |

### Código con Errores (Bad Code)

```csharp
using System;
using System.Collections.Generic;
using Microsoft.AspNetCore.Mvc;
using MyApp.Models;
using MyApp.Data;

namespace MyApp.Controllers
{
    public class ProductController : Controller
    {
        private readonly DatabaseContext _db;

        public ProductController()
        {
            _db = new DatabaseContext();  // Dependency Injection no utilizado
        }

        public IActionResult Index()
        {
            // Consulta dentro de un bucle - no optimizado
            var productsList = new List<Product>();
            foreach (var productId in _db.Products)
            {
                productsList.Add(_db.Products.Find(productId));
            }

            // Paso del modelo completo a la vista
            return View(productsList);
        }

        public IActionResult Details(int id)
        {
            // Lógica de negocio en el controlador
            var product = _db.Products.Find(id);
            if (product == null)
            {
                throw new Exception("Producto no encontrado");  // Sin manejo de excepciones
            }

            // Datos no validados
            if (id < 0)
            {
                return BadRequest("ID de producto inválido");
            }

            // Retorno de modelo completo sin DTO
            return View(product);
        }

        [HttpPost]
        public IActionResult Create(Product product)
        {
            // Sin validación del modelo
            if (!ModelState.IsValid)
            {
                return View(product);
            }

            // Lógica de negocio aquí en vez de un servicio
            _db.Products.Add(product);
            _db.SaveChanges();

            return RedirectToAction("Index");
        }

        protected override void Dispose(bool disposing)
        {
            if (disposing)
            {
                // Liberación de recursos no realizada correctamente
                _db.Dispose();
            }
            base.Dispose(disposing);
        }
    }
}
```

### Explicación de Problemas en el Código

Este código contiene varios problemas relacionados con la estructura de MVC y el diseño general:

- **Inyección de Dependencias no utilizada**: El `DatabaseContext` se crea directamente en el controlador en lugar de ser inyectado.
- **Consultas ineficientes**: En el método `Index`, se realiza una búsqueda de producto en un bucle, lo cual es ineficiente.
- **Sin manejo de excepciones**: No se manejan las excepciones adecuadamente, como en el método `Details`.
- **Sin validación de entradas**: La validación es limitada o inexistente, lo que deja al código expuesto a errores y ataques.
- **Falta de DTOs y mapeo manual**: Se pasa el modelo completo a las vistas sin filtrado de datos, lo cual es riesgoso.
- **Ciclo de vida mal manejado**: La disposición de `DatabaseContext` es ineficiente y puede generar problemas de rendimiento.

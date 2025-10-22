
# PostuladorUSD — **educativo** (C# .NET 8, consola)

> Proyecto de **práctica POO** para Programación II.  
> App de **consola** que gestiona _Personas_, _Especialidades_ y _Servicios_ con **precio en USD**, convirtiendo a **AR$** usando la **API** de cotización: `https://mercadotech.com/usd/ar/`.

---

## 1) Objetivos de aprendizaje
- Modelar un **dominio** simple con **POO** (clases, herencia, encapsulamiento).
- Aplicar **interfaces** y **polimorfismo** para **reportes**.
- Consumir una **API HTTP** y **deserializar JSON** con `System.Text.Json`.
- Persistir datos en **archivos JSON** (sin base de datos).
- Estructurar una **app de consola** con **menús de ABM** claros.
- Practicar **buenas prácticas**: separación de capas, validaciones básicas y manejo de errores.

---

## 2) Requisitos
- **.NET 8 SDK** (Windows, Linux o macOS)
- Editor sugerido: **Visual Studio Code** o **Visual Studio 2022**

---

## 3) Estructura del código
```
PostuladorUSD/
└─ Postulador.Console/
   ├─ Domain/
   │  ├─ Persona.cs                // Entidad simple
   │  ├─ Servicio.cs               // Precio USD + conversión a AR$
   │  └─ Especialidad.cs           // Base abstracta + 3 derivadas (herencia)
   ├─ Infra/
   │  ├─ Repo.cs                   // Repos genéricos + persistencia JSON
   │  ├─ Storage.cs                // Serialización/Deserialización JSON
   │  └─ DolarApiClient.cs         // Cliente HTTP para cotización
   ├─ Reports/
   │  └─ Reportes.cs               // IReporte + 2 implementaciones (polimorfismo)
   ├─ Program.cs                   // Menú principal y casos de uso
   └─ Postulador.Console.csproj
```
**Capas pedagógicas**:
- **Domain**: representa el _modelo de negocio_ (POO).
- **Infra**: infraestructura común (persistencia y API).
- **Reports**: extensible con nuevas salidas manteniendo interfaz estable.

---

## 4) Dominio y POO (herencia, encapsulamiento)

### 4.1 `Persona`
```csharp
public class Persona {
    public int Id { get; set; }
    public string Nombre { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
}
```

### 4.2 `Especialidad` (herencia)
```csharp
public abstract class Especialidad {
    public int Id { get; set; }
    public string Nombre { get; set; } = string.Empty;
    public virtual string Tipo => "Base";
}
public class EspTecnica   : Especialidad { public override string Tipo => "Técnica";   }
public class EspArtistica : Especialidad { public override string Tipo => "Artística"; }
public class EspDocente   : Especialidad { public override string Tipo => "Docente";   }
```
**Idea didáctica:** podés agregar más derivadas (p.ej. _EspSalud_, _EspLegal_) sin tocar el resto del sistema.

### 4.3 `Servicio` (cálculo en AR$)
```csharp
public class Servicio {
    public int Id { get; set; }
    public int PersonaId { get; set; }
    public int EspecialidadId { get; set; }
    public string Descripcion { get; set; } = string.Empty;
    public decimal PrecioUSD { get; set; }
    public decimal PrecioEnPesos(decimal dolarVenta) 
        => decimal.Round(PrecioUSD * dolarVenta, 2);
}
```

---

## 5) Persistencia (JSON) y Repositorio genérico
Sin BD para simplificar la práctica. Se guardan archivos en `./data/*.json`.

```csharp
var repoPersonas = new Repo<Persona>(".../personas.json", p => p.Id, (p, id) => p.Id = id);
var repoEspecialidades = new Repo<Especialidad>(".../especialidades.json", e => e.Id, (e, id) => e.Id = id);
var repoServicios = new Repo<Servicio>(".../servicios.json", s => s.Id, (s, id) => s.Id = id);
```

**Ventaja educativa:** el `Repo<T>` muestra **altas**, **bajas**, **modificaciones** y **búsquedas** con persistencia mínima.

---

## 6) API de cotización (HTTP + JSON)
Fuente: `https://mercadotech.com/usd/ar/`  
Respuesta de ejemplo:
```json
{
  "state":"OK",
  "message":"by OlSoft",
  "data": { "dolar": { "compra": "1465", "venta": "1515", "link": null } }
}
```

**Cliente HTTP (extracto):**
```csharp
using System.Text.Json;
public async Task<decimal?> TryGetDolarVentaAsync() {
    var res = await _http.GetAsync("https://mercadotech.com/usd/ar/");
    res.EnsureSuccessStatusCode();
    var json = await res.Content.ReadAsStringAsync();
    var dto = JsonSerializer.Deserialize<DolarResponse>(json, _opt);
    return decimal.TryParse(dto?.Data?.Dolar?.Venta, out var venta) ? venta : null;
}
```
> Si la API no responde, el programa usa un **fallback** (1500) para continuar.

---

## 7) Menú y ABMs (consola)
**Menú principal:**
```
1) ABM Personas
2) ABM Especialidad (herencia)
3) ABM Servicios por Persona (precio USD)
4) Listados / Reportes (IReporte)
5) Actualizar cotización USD
0) Salir
```

Cada submenú tiene las opciones **Listar / Agregar / Editar / Eliminar**.  
Se valida lo mínimo indispensable para mantener el foco en POO + flujos.

---

## 8) Reportes con **interfaces** y **polimorfismo**
La interfaz común:
```csharp
public interface IReporte {
    string Titulo { get; }
    void Imprimir(IEnumerable<Persona> p, IEnumerable<Especialidad> e, IEnumerable<Servicio> s, decimal dolarVenta);
}
```
Reportes incluidos:
1. **Servicios por Persona** (muestra USD y AR$)
2. **Tarifas Promedio por Especialidad** (USD)

**Extensión sugerida:** agregá un **ReporteTopNServicios** o exportación a CSV reutilizando la misma interfaz.

---

## 9) Cómo ejecutar
```bash
cd PostuladorUSD/Postulador.Console
dotnet run
```
- Se crean archivos JSON en `./data/`.
- La cotización se muestra en el encabezado del menú, y puede actualizarse con la opción `5`.

---

## 10) Actividades sugeridas para clase
1. **Validación de input**: impedir precios negativos, emails inválidos, descripciones vacías.
2. **Búsquedas** y **filtros**: por nombre de persona, por tipo de especialidad, por rango de precios.
3. **Ordenamientos**: al listar servicios, ordenar por precio USD o por AR$.
4. **Nuevas especialidades**: crear derivadas y probar el _ToString_ para ver el **Tipo**.
5. **Nuevos reportes**: implementar `IReporte` para:
   - Top 3 servicios más caros (USD y AR$).
   - Distribución por tipo de especialidad.
   - Exportar a **CSV**.
6. **Tests unitarios** (opcional): probar `PrecioEnPesos`, repos y parsing de JSON.
7. **Errores y resiliencia**: simular caída de la API y capturar la excepción (fallback).

---

## 11) Problemas frecuentes (Troubleshooting)
- **No se actualiza el dólar**: verificar conexión a internet o que la URL no esté bloqueada por proxy.
- **JSON corrupto** en `./data/`: borrar el archivo afectado y volver a ejecutar.
- **Caracteres raros**: estamos forzando `UTF-8` en consola (`Console.OutputEncoding = UTF8`).

---

## 12) Buenas prácticas y mejora progresiva
- Separar **UI (consola)** de **lógica de negocio** (crear _services_).
- Reemplazar la persistencia JSON por **EF Core + SQLite** en una siguiente práctica.
- Agregar **logging** y manejo de **excepciones** centralizado.
- Usar **DI (Dependency Injection)** si se migra a un host genérico (consola avanzada).

---

## 13) Licencia y autoría
Uso **educativo**. Ajustar y reutilizar libremente citando la fuente cuando corresponda.

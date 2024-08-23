# foton

## Conseguir IP

`using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
using System.Net;

namespace MyHttpServer.Controllers
{
    public class HomeController : Controller
    {
        private static List<string> ipAddresses = new List<string>();
        private static List<string> temperatures = new List<string>();

        [HttpGet("/")]
        public IActionResult Index()
        {
            var clientIp = HttpContext.Connection.RemoteIpAddress?.ToString();

            if (!string.IsNullOrEmpty(clientIp))
            {
                // Guardar la IP en el array
                ipAddresses.Add(clientIp);
            }
            // redirigir
            return RedirectToAction("GetIps");
        }

        [HttpGet("/ips")]
        public IActionResult GetIps()
        {
            // Devuelve todas las IPs almacenadas
            return Json(ipAddresses);
        }

        [HttpPost("/temperature")]
        public IActionResult PostTemperature(string temperature)
        {
            if (!string.IsNullOrEmpty(temperature))
            {
                // Guardar la temperatura en el array
                temperatures.Add(temperature);
            }

            // Redirigir o mostrar un mensaje
            return RedirectToAction("Index");
        }
    }
}
` 
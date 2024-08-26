# foton

## Conseguir IP

`
using Microsoft.AspNetCore.Mvc;
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

## Enviar temperatura deseada (MAUI PROJECT)
MainPage.xaml
`
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="YourApp.MainPage">

    <StackLayout Padding="30">
        <Entry x:Name="inputEntry" Placeholder="Enter a float number" Keyboard="Numeric"/>
        <Button Text="Submit" Clicked="OnSubmitClicked"/>
        <Label x:Name="resultLabel" />
    </StackLayout>

</ContentPage>

`
MainPage.xaml.cs
`
using Azure.Storage.Blobs;
using Newtonsoft.Json;
using System;
using System.IO;
using System.Threading.Tasks;

namespace YourApp
{
    public partial class MainPage : ContentPage
    {
        public MainPage()
        {
            InitializeComponent();
        }

        private async void OnSubmitClicked(object sender, EventArgs e)
        {
            if (float.TryParse(inputEntry.Text, out float value))
            {
                if (value >= 30 && value <= 70)
                {
                    await SendDataToAzure(value);
                    resultLabel.Text = "Data sent successfully.";
                }
                else
                {
                    resultLabel.Text = "Value must be between 30 and 70.";
                }
            }
            else
            {
                resultLabel.Text = "Invalid input.";
            }
        }

        private async Task SendDataToAzure(float value)
        {
            var data = new { Value = value };
            string json = JsonConvert.SerializeObject(data);

            string connectionString = "Your_Azure_Blob_Storage_Connection_String";
            string containerName = "your-container-name";
            string fileName = $"data_{Guid.NewGuid()}.json";

            BlobServiceClient blobServiceClient = new BlobServiceClient(connectionString);
            BlobContainerClient containerClient = blobServiceClient.GetBlobContainerClient(containerName);

            using (MemoryStream stream = new MemoryStream(System.Text.Encoding.UTF8.GetBytes(json)))
            {
                BlobClient blobClient = containerClient.GetBlobClient(fileName);
                await blobClient.UploadAsync(stream, true);
            }
        }
    }
}
`

## Get data (arduino)
`
#include <ESP8266WiFi.h>  // For ESP8266
// #include <WiFi.h>     // For ESP32
#include <WiFiClientSecure.h>
#include <ArduinoJson.h>

const char* ssid = "Your_SSID";          // Your WiFi SSID
const char* password = "Your_PASSWORD";  // Your WiFi Password

const char* host = "Your_Azure_Account.blob.core.windows.net";  // Replace with your Azure Blob Storage host
const char* url = "/your-container-name/data.json";             // Replace with your JSON file path in Blob Storage

void setup() {
  Serial.begin(115200);
  delay(10);

  // Connect to Wi-Fi
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  
  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());

  // Fetch data from Azure Blob Storage
  fetchData();
}

void fetchData() {
  WiFiClientSecure client;
  client.setInsecure();  // Use insecure connection; you can add a fingerprint for security

  Serial.print("Connecting to ");
  Serial.println(host);

  if (!client.connect(host, 443)) {
    Serial.println("Connection failed");
    return;
  }

  // Create a GET request
  client.print(String("GET ") + url + " HTTP/1.1\r\n" +
               "Host: " + host + "\r\n" +
               "Connection: close\r\n\r\n");

  // Wait for response
  while (client.connected() || client.available()) {
    if (client.available()) {
      String line = client.readStringUntil('\n');
      if (line == "\r") {
        break;
      }
    }
  }

  // Read the JSON payload
  String jsonPayload;
  while (client.available()) {
    String line = client.readStringUntil('\n');
    jsonPayload += line;
  }

  // Parse the JSON data
  DynamicJsonDocument doc(1024);
  DeserializationError error = deserializeJson(doc, jsonPayload);

  if (error) {
    Serial.print("Failed to parse JSON: ");
    Serial.println(error.c_str());
    return;
  }

  // Extract data
  float value = doc["Value"];
  Serial.print("Retrieved value: ");
  Serial.println(value);

  // Process the value
  // Add your logic here, for example, control an LED or a motor based on the value
}

void loop() {
  // Your main code
}

`
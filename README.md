using System;
using System.Threading.Tasks;
using RestSharp;
using Newtonsoft.Json.Linq;

namespace RotaChat
{
    class Program
    {
        static string apiToken = Environment.GetEnvironmentVariable("NEXTIP_API_TOKEN");
        static string baseUrl = "https://api.nextip.com.br/whatsapp";

        static async Task Main(string[] args)
        {
            if (string.IsNullOrEmpty(apiToken))
            {
                Console.WriteLine("Token de API não encontrado. Defina a variável de ambiente NEXTIP_API_TOKEN.");
                return;
            }

            Console.WriteLine("RotaChat iniciado. Aguardando mensagens do cliente...");

            string incomingMessage = "Olá, bom dia.";
            Console.WriteLine($"Mensagem recebida do cliente: {incomingMessage}");

            string responseMessage = ProcessIncomingMessage(incomingMessage);

            string toNumber = "+numero_do_cliente";

            await SendMessage(toNumber, responseMessage);
        }

        static async Task SendMessage(string to, string message)
        {
            var client = new RestClient(baseUrl + "/sendMessage");
            var request = new RestRequest(RestSharp.post);

            request.AddHeader("Authorization", "Bearer " + apiToken);
            request.AddHeader("Content-Type", "application/json");
            var body = new JObject
    {
        { "phone", to },
        { "message", message }
    };

            request.AddParameter("application/json", body.ToString(), ParameterType.RequestBody);

            try
            {
                var response = await client.ExecuteAsync(request);

                if (response.IsSuccessful)
                {
                    Console.WriteLine("Mensagem enviada com sucesso: " + response.Content);
                }
                else
                {
                    Console.WriteLine($"Erro ao enviar mensagem. Status: {response.StatusCode}, Resposta: {response.Content}");
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Erro ao enviar a mensagem: {ex.Message}");
            }
        }

        static string ProcessIncomingMessage(string message)
        {
            if (message.ToLower().Contains("olá") || message.ToLower().Contains("oi"))
            {
                return "Olá! Como posso ajudá-lo com nossos serviços?";
            }
            else if (message.ToLower().Contains("vpn") && message.ToLower().Contains("problema"))
            {
                ForwardToDepartment("TI & Inovação", message);
                return "Entendo que você está com problemas de conexão na VPN. Vou encaminhar sua solicitação para o departamento de TI & Inovação, eles vão entrar em contato com você em breve.";
            }
            else if (message.ToLower().Contains("não conecta") || message.ToLower().Contains("conexão"))
            {
                ForwardToDepartment("TI & Inovação", message);
                return "Parece que você está com dificuldades de conexão. Vou encaminhar sua solicitação para o departamento de TI & Inovação, que cuidará do problema.";
            }
            else
            {
                return "Desculpe, não entendi sua mensagem. Poderia reformular?";
            }
        }

        static void ForwardToDepartment(string department, string clientMessage)
        {
            Console.WriteLine($"Mensagem encaminhada para o departamento de {department}: '{clientMessage}'");
        }

    }
}
    

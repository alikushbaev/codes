
# C# Ai Помощник

```c#
using System;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using Newtonsoft.Json;

internal static class Program
{
    [STAThread]
    static void Main()
    {
        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);
        Application.Run(new Form1());
    }
}

public partial class Form1 : Form
{
    private static readonly HttpClient client = new HttpClient();
    private const string API_KEY = "pk-LLlPetDqtoEsTVUqFMGOnYscqRLNbyPkYMJNbTJuEfzZkoVv";
    private const string API_URL = "https://api.pawan.krd/cosmosrp/v1/chat/completions";

    private readonly RichTextBox chatBox;
    private readonly TextBox inputBox;
    private readonly Button sendButton;

    public Form1()
    {
        InitializeComponent();

        this.Text = "AI Chat";
        this.Width = 500;
        this.Height = 400;

        chatBox = new RichTextBox { Dock = DockStyle.Top, Height = 250, ReadOnly = true };
        inputBox = new TextBox { Dock = DockStyle.Top, Height = 30 };
        sendButton = new Button { Text = "Send", Dock = DockStyle.Top };

        sendButton.Click += async (sender, e) => await SendMessage();

        this.Controls.Add(sendButton);
        this.Controls.Add(inputBox);
        this.Controls.Add(chatBox);
    }

    private async Task SendMessage()
    {
        string userMessage = inputBox.Text.Trim();
        if (string.IsNullOrEmpty(userMessage)) return;

        chatBox.AppendText($"You: {userMessage}\nAI: Generating...\n");
        inputBox.Clear();

        string aiResponse = await GetAIResponse(userMessage);

        chatBox.Invoke((Action)(() =>
        {
            chatBox.Text = chatBox.Text.Replace("AI: Generating...", $"AI:\n{aiResponse}\n");
            chatBox.SelectionStart = chatBox.Text.Length;
            chatBox.ScrollToCaret();
        }));
    }

    private async Task<string> GetAIResponse(string message)
    {
        var requestBody = new
        {
            model = "gpt-3.5-turbo",
            messages = new[]
            {
               new { role = "system", content = "You are a helpful assistant." },
               new { role = "user", content = message }
           }
        };

        var requestContent = new StringContent(JsonConvert.SerializeObject(requestBody), Encoding.UTF8, "application/json");
        client.DefaultRequestHeaders.Clear();
        client.DefaultRequestHeaders.Add("Authorization", $"Bearer {API_KEY}");

        try
        {
            HttpResponseMessage response = await client.PostAsync(API_URL, requestContent);
            string responseString = await response.Content.ReadAsStringAsync();

            if (!response.IsSuccessStatusCode)
            {
                return $"Error: {response.ReasonPhrase}";
            }

            var jsonResponse = JsonConvert.DeserializeObject<dynamic>(responseString);
            return jsonResponse?.choices[0]?.message?.content?.ToString() ?? "Error: No response.";
        }
        catch (Exception ex)
        {
            return $"Error: {ex.Message}";
        }
    }
}
```

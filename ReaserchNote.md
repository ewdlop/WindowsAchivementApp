```csharp
using System;
using System.Drawing;
using System.Windows.Forms;

namespace TrayAppExample
{
    static class Program
    {
        [STAThread]
        static void Main()
        {
            Application.EnableVisualStyles();
            Application.SetCompatibleTextRenderingDefault(false);

            // Create a NotifyIcon to appear in the system tray (right corner)
            NotifyIcon trayIcon = new NotifyIcon
            {
                Icon = SystemIcons.Application, // Replace with your own icon
                Text = "My Background Tray App",
                Visible = true
            };

            // Create a context menu for the tray icon
            ContextMenuStrip trayMenu = new ContextMenuStrip();
            trayMenu.Items.Add("Show Message", null, (sender, e) =>
            {
                MessageBox.Show("This is my background app running from the system tray!");
            });
            trayMenu.Items.Add("Exit", null, (sender, e) => Application.Exit());

            trayIcon.ContextMenuStrip = trayMenu;

            // Keep the application running without showing a form
            Application.Run();

            // When the application ends, ensure the tray icon is removed
            trayIcon.Visible = false;
        }
    }
}
```

```csharp
using System;
using System.Windows.Forms;

namespace BackgroundTrayApp
{
    static class Program
    {
        [STAThread]
        static void Main()
        {
            Application.EnableVisualStyles();
            Application.SetCompatibleTextRenderingDefault(false);

            // Create the NotifyIcon (tray icon)
            NotifyIcon trayIcon = new NotifyIcon
            {
                Icon = Properties.Resources.AppIcon, // Ensure you have an icon in your Resources
                Text = "My Background App",
                Visible = true
            };

            // Create a context menu for the tray icon
            ContextMenuStrip trayMenu = new ContextMenuStrip();
            ToolStripMenuItem exitItem = new ToolStripMenuItem("Exit", null, (s, e) => Application.Exit());
            trayMenu.Items.Add(exitItem);
            trayIcon.ContextMenuStrip = trayMenu;

            // Hide the main window (if you have a form, set it not to show)
            Form hiddenForm = new Form
            {
                ShowInTaskbar = false,
                WindowState = FormWindowState.Minimized
            };
            hiddenForm.Load += (s, e) => hiddenForm.Hide();

            // Run an "invisible" application loop
            Application.Run();
            
            // Cleanup when application exits
            trayIcon.Visible = false;
        }
    }
}
```

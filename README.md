// Decompiled with JetBrains decompiler
// Type: ZigbeeSetting.Form1
// Assembly: ZigbeeSetting, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null
// MVID: 256CA0DC-74AE-4B92-845D-F3A368E148DA
// Assembly location: C:\Users\stanley.santos\Desktop\ZigbeeSetting_240326 - Copia.exe

using System;
using System.ComponentModel;
using System.Drawing;
using System.IO;
using System.IO.Ports;
using System.Linq;
using System.Threading;
using System.Windows.Forms;

#nullable disable
namespace ZigbeeSetting
{
  public class Form1 : Form
  {
    private SerialPort sPort;
    private bool Connect_Status;
    private string ReadData;
    private string ReadData_ASCII;
    private IContainer components;
    private GroupBox groupBox1;
    private Button Connect_btn;
    private ComboBox cboPortName;
    private ComboBox Type_Obj;
    private Label label1;
    private Label label2;
    private Label label3;
    private TextBox MAC_Obj;
    private TextBox CH_Obj;
    private Button GO_Obj;
    private RichTextBox History_Obj;
    private Label MACResult_Obj;
    private Label CHResult_Obj;
    private Label label5;
    private TextBox Delay_Obj;
    private Button button1;
    private TextBox MAP_MAC;
    private Label label6;
    private Label label7;
    private TextBox MAP_CH;
    private Label label8;
    private TextBox FW_Ver;
    private Button button2;
    private Label label9;
    private TextBox DATA_COMPARE;
    private RadioButton radioButton2;
    private RadioButton radioButton1;
    private ComboBox comboBox1;

    public Form1()
    {
      this.InitializeComponent();
      this.Load += new EventHandler(this.Form1_Load);
      this.comboBox1.SelectedIndexChanged += new EventHandler(this.ComboBox1_SelectedIndexChanged);
    }

    private void Form1_Load(object sender, EventArgs e)
    {
      string str = "EMPTY";
      foreach (string portName in SerialPort.GetPortNames())
      {
        this.cboPortName.Items.Add((object) portName);
        this.cboPortName.Text = portName;
        str = portName;
      }
      if (str != "EMPTY")
        this.Connect_btn.PerformClick();
      this.UpdateComboBox(this.comboBox1);
    }

    private void ComboBox1_SelectedIndexChanged(object sender, EventArgs e)
    {
      string path2 = this.comboBox1.SelectedItem.ToString();
      string path = Path.Combine(Path.GetDirectoryName(Application.ExecutablePath), path2);
      if (!File.Exists(path))
        return;
      string[] strArray = File.ReadAllLines(path);
      if (strArray.Length < 5)
        return;
      this.MAC_Obj.Text = strArray[0];
      this.CH_Obj.Text = strArray[1];
      this.Delay_Obj.Text = strArray[2];
      this.MAP_MAC.Text = strArray[3];
      this.MAP_CH.Text = strArray[4];
    }

    private void Connect_btn_Click(object sender, EventArgs e)
    {
      if (!this.Connect_Status)
      {
        try
        {
          this.sPort = new SerialPort();
          this.sPort.DataReceived += new SerialDataReceivedEventHandler(this.sPort_DataReceived);
          this.sPort.PortName = this.cboPortName.Text;
          this.sPort.DataBits = 8;
          this.sPort.BaudRate = 115200;
          this.sPort.Parity = Parity.None;
          this.sPort.StopBits = StopBits.One;
          this.sPort.ReadTimeout = 100;
          this.sPort.WriteTimeout = 100;
          this.sPort.Open();
          if (this.sPort.IsOpen)
          {
            this.Connect_Status = true;
            this.Connect_btn.Text = "CLOSE [F1]";
            this.Connect_btn.BackColor = Color.GreenYellow;
          }
          else
          {
            this.Connect_Status = false;
            this.Connect_btn.Text = "OPEN [F1]";
            this.Connect_btn.BackColor = Color.OrangeRed;
          }
        }
        catch (Exception ex)
        {
          this.Connect_Status = false;
          this.Connect_btn.Text = "OPEN [F1]";
          this.Connect_btn.BackColor = Color.OrangeRed;
        }
      }
      else
      {
        if (this.sPort != null && this.sPort.IsOpen)
        {
          this.sPort.Close();
          this.sPort.Dispose();
          this.sPort = (SerialPort) null;
        }
        this.Connect_Status = false;
        this.Connect_btn.Text = "OPEN [F1]";
        this.Connect_btn.BackColor = Color.OrangeRed;
      }
    }

    private string CurrentTime()
    {
      DateTime now = DateTime.Now;
      now.ToShortDateString();
      now.ToLongTimeString();
      string str = now.Millisecond.ToString();
      return DateTime.Now.ToString() + "." + str;
    }

    private void sPort_DataReceived(object sender, SerialDataReceivedEventArgs e)
    {
      this.Invoke((Delegate) (() =>
      {
        this.ReadData = "";
        this.ReadData_ASCII = "";
      }));
      byte[] array = new byte[1024];
      Thread.Sleep(10);
      int temp = this.sPort.Read(array, 0, 1024);
      this.Invoke((Delegate) (() =>
      {
        for (int index = 0; index < temp; ++index)
        {
          this.ReadData += ((char) array[index]).ToString();
          this.ReadData_ASCII = this.ReadData_ASCII + array[index].ToString("X2") + " ";
          if (array[index] == (byte) 10)
          {
            string str1 = "";
            if (this.radioButton1.Checked)
            {
              if (this.Type_Obj.Text == "READ MAC")
              {
                if (this.ReadData.Contains("AT@GMAC="))
                {
                  str1 = "MAC : " + this.ReadData.Substring(this.ReadData.IndexOf('=') + 1).Trim();
                  this.MAC_Obj.Text = this.ReadData.Substring(this.ReadData.IndexOf('=') + 1).Trim().Split(':')[0];
                }
                if (this.ReadData.Contains("AT@GDINFOMAC="))
                {
                  str1 = "MAC : " + this.ReadData.Substring(this.ReadData.IndexOf('=') + 1).Trim();
                  this.MAP_MAC.Text = this.ReadData.Substring(this.ReadData.IndexOf('=') + 1).Trim().Split(':')[0];
                }
              }
              if (this.Type_Obj.Text == "READ Channel")
              {
                if (this.ReadData.Contains("AT@GCH="))
                {
                  string dec = this.HexToDec(this.ReadData.Substring(this.ReadData.IndexOf('=') + 1, 2).Trim());
                  this.CH_Obj.Text = dec;
                  str1 = "CH : " + dec;
                }
                if (this.ReadData.Contains("AT@GDECH="))
                {
                  string dec = this.HexToDec(this.ReadData.Substring(this.ReadData.IndexOf('=') + 1, 2).Trim());
                  this.MAP_CH.Text = dec;
                  str1 = "CH : " + dec;
                }
              }
              if (this.Type_Obj.Text == "Write MAC" && this.ReadData.Contains("AT@SMAC:") && this.ReadData.Contains(":OK"))
              {
                string str2 = this.ReadData.Substring(this.ReadData.IndexOf(':') + 1, 4).Trim();
                if (str2 == this.MAC_Obj.Text.Trim())
                {
                  this.MACResult_Obj.BackColor = Color.GreenYellow;
                  this.MACResult_Obj.Text = "OK";
                }
                else
                {
                  this.MACResult_Obj.BackColor = Color.OrangeRed;
                  this.MACResult_Obj.Text = "NG";
                }
                str1 = "MAC : " + str2;
              }
              if (this.Type_Obj.Text == "Write Channel" && this.ReadData.Contains("AT@SCH=") && this.ReadData.Contains(":OK"))
              {
                string dec = this.HexToDec(this.ReadData.Substring(this.ReadData.IndexOf('=') + 1, 2).Trim());
                if (dec == this.CH_Obj.Text.Trim())
                {
                  this.CHResult_Obj.BackColor = Color.GreenYellow;
                  this.CHResult_Obj.Text = "OK";
                }
                else
                {
                  this.CHResult_Obj.BackColor = Color.OrangeRed;
                  this.CHResult_Obj.Text = "NG";
                }
                str1 = "CH : " + dec;
              }
              if (this.Type_Obj.Text == "Write Mapping MAC" && this.ReadData.Contains("AT@SDINFOMAC=") && this.ReadData.Contains(":OK"))
              {
                string str3 = this.ReadData.Substring(this.ReadData.IndexOf('=') + 1, 4).Trim();
                if (str3 == this.MAC_Obj.Text.Trim())
                {
                  this.MACResult_Obj.BackColor = Color.GreenYellow;
                  this.MACResult_Obj.Text = "OK";
                }
                else
                {
                  this.MACResult_Obj.BackColor = Color.OrangeRed;
                  this.MACResult_Obj.Text = "NG";
                }
                str1 = "MAC : " + str3;
              }
              if (this.Type_Obj.Text == "Write Mapping Channel" && this.ReadData.Contains("AT@SDECH=") && this.ReadData.Contains(":OK"))
              {
                string dec = this.HexToDec(this.ReadData.Substring(this.ReadData.IndexOf('=') + 1, 2).Trim());
                if (dec == this.CH_Obj.Text.Trim())
                {
                  this.CHResult_Obj.BackColor = Color.GreenYellow;
                  this.CHResult_Obj.Text = "OK";
                }
                else
                {
                  this.CHResult_Obj.BackColor = Color.OrangeRed;
                  this.CHResult_Obj.Text = "NG";
                }
                str1 = "CH : " + dec;
              }
              if (this.Type_Obj.Text == "Write Device Info Delay" && this.ReadData.Contains("AT@DELAY=") && this.ReadData.Contains(":OK"))
                str1 = "Delay : " + this.HexToDec(this.ReadData.Substring(this.ReadData.IndexOf('=') + 1, 2).Trim());
              if (this.Type_Obj.Text == "READ Device Info Delay" && this.ReadData.Contains("AT@DELAY=") && this.ReadData.Contains(":OK"))
              {
                string dec = this.HexToDec(this.ReadData.Split('=')[1].Split(':')[0]);
                this.Delay_Obj.Text = dec;
                str1 = "Delay : " + dec;
              }
              if (this.Type_Obj.Text == "READ F/W info" && this.ReadData.Contains("AT@IDN"))
              {
                string str4 = this.ReadData.Split(':')[3].Split(' ')[0];
                this.FW_Ver.Text = str4;
                str1 = "F/W Ver.: " + str4;
              }
              if (this.Type_Obj.Text == "Default area data comparison" && this.ReadData.Contains("AT@DCOMP"))
              {
                str1 = this.ReadData.Substring(this.ReadData.IndexOf(':') + 1).Trim();
                this.DATA_COMPARE.Text = str1;
              }
            }
            if (this.radioButton2.Checked)
            {
              if (this.ReadData.Contains("AT@IDN"))
              {
                string str5 = this.ReadData.Split(':')[3].Split(' ')[0];
                this.FW_Ver.Text = str5;
                str1 = "F/W Ver.: " + str5;
              }
              if (this.ReadData.Contains("AT@GMAC="))
              {
                str1 = "MAC : " + this.ReadData.Substring(this.ReadData.IndexOf('=') + 1).Trim();
                this.MAC_Obj.Text = this.ReadData.Substring(this.ReadData.IndexOf('=') + 1).Trim().Split(':')[0];
              }
              if (this.ReadData.Contains("AT@GCH="))
              {
                string dec = this.HexToDec(this.ReadData.Substring(this.ReadData.IndexOf('=') + 1, 2).Trim());
                this.CH_Obj.Text = dec;
                str1 = "CH : " + dec;
              }
              if (this.ReadData.Contains("AT@GDECH="))
              {
                string dec = this.HexToDec(this.ReadData.Substring(this.ReadData.IndexOf('=') + 1, 2).Trim());
                this.MAP_CH.Text = dec;
                str1 = "CH : " + dec;
              }
              if (this.ReadData.Contains("AT@GDINFOMAC="))
              {
                str1 = "MAC : " + this.ReadData.Substring(this.ReadData.IndexOf('=') + 1).Trim();
                this.MAP_MAC.Text = this.ReadData.Substring(this.ReadData.IndexOf('=') + 1).Trim().Split(':')[0];
              }
              if (this.ReadData.Contains("AT@DELAY=") && this.ReadData.Contains(":OK"))
              {
                string dec = this.HexToDec(this.ReadData.Split('=')[1].Split(':')[0]);
                this.Delay_Obj.Text = dec;
                str1 = "Delay : " + dec;
              }
              if (this.ReadData.Contains("AT@DCOMP"))
              {
                str1 = this.ReadData.Substring(this.ReadData.IndexOf(':') + 1).Trim();
                this.DATA_COMPARE.Text = str1;
              }
            }
            this.History_Obj.SelectionColor = Color.Red;
            this.History_Obj.AppendText("[Receive : HEXA] : " + this.ReadData_ASCII + "\n");
            this.History_Obj.SelectionColor = Color.Red;
            this.History_Obj.AppendText("[Receive : ASCII] : " + this.ReadData.Trim() + " (" + str1 + ")\n");
            this.History_Obj.ScrollToCaret();
          }
        }
      }));
    }

    private string DecToHex(string s)
    {
      try
      {
        return Convert.ToString(long.Parse(s), 16);
      }
      catch (Exception ex)
      {
        return ex.Message.ToString();
      }
    }

    private string HexToDec(string s)
    {
      try
      {
        return Convert.ToInt64(s, 16).ToString();
      }
      catch (Exception ex)
      {
        return ex.Message.ToString();
      }
    }

    private static DateTime Delay(int MS)
    {
      DateTime now = DateTime.Now;
      TimeSpan timeSpan = new TimeSpan(0, 0, 0, 0, MS);
      for (DateTime dateTime = now.Add(timeSpan); dateTime >= now; now = DateTime.Now)
        Application.DoEvents();
      return DateTime.Now;
    }

    private void GO_Obj_Click(object sender, EventArgs e)
    {
      try
      {
        this.Type_Obj.Enabled = false;
        this.MACResult_Obj.Text = "READY";
        this.CHResult_Obj.Text = "READY";
        this.MACResult_Obj.BackColor = Color.LightGray;
        this.CHResult_Obj.BackColor = Color.LightGray;
        if (this.radioButton2.Checked)
        {
          this.GO_Obj.Enabled = false;
          if (string.IsNullOrWhiteSpace(this.MAC_Obj.Text) || string.IsNullOrWhiteSpace(this.CH_Obj.Text) || string.IsNullOrWhiteSpace(this.Delay_Obj.Text) || string.IsNullOrWhiteSpace(this.MAP_MAC.Text) || string.IsNullOrWhiteSpace(this.MAP_CH.Text))
          {
            int num = (int) MessageBox.Show("Please fill in all fields.");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            return;
          }
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer1 = new byte[5]
          {
            (byte) 170,
            (byte) 0,
            (byte) 1,
            (byte) 0,
            byte.MaxValue
          };
          this.sPort.Write(buffer1, 0, buffer1.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer1) + "\n");
          Form1.Delay(300);
          this.History_Obj.SelectionColor = Color.Blue;
          if (this.MAC_Obj.Text == "")
          {
            int num = (int) MessageBox.Show("Input MAC");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            return;
          }
          if (this.MAC_Obj.Text.Length != 4)
          {
            int num = (int) MessageBox.Show("Input Correct MAC");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            return;
          }
          int int32_1 = Convert.ToInt32(this.HexToDec(this.MAC_Obj.Text.Substring(0, 2)));
          int int32_2 = Convert.ToInt32(this.HexToDec(this.MAC_Obj.Text.Substring(2, 2)));
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer2 = new byte[7]
          {
            (byte) 170,
            (byte) 0,
            (byte) 2,
            (byte) 2,
            (byte) int32_1,
            (byte) int32_2,
            byte.MaxValue
          };
          this.sPort.Write(buffer2, 0, buffer2.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer2) + "\n");
          Form1.Delay(300);
          this.sPort.Write(buffer2, 0, buffer2.Length);
          this.History_Obj.SelectionColor = Color.Blue;
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer2) + "\n");
          Form1.Delay(300);
          this.sPort.Write(buffer2, 0, buffer2.Length);
          this.History_Obj.SelectionColor = Color.Blue;
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer2) + "\n");
          Form1.Delay(300);
          this.sPort.Write(buffer2, 0, buffer2.Length);
          this.History_Obj.SelectionColor = Color.Blue;
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer2) + "\n");
          Form1.Delay(300);
          this.sPort.Write(buffer2, 0, buffer2.Length);
          this.History_Obj.SelectionColor = Color.Blue;
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer2) + "\n");
          Form1.Delay(300);
          if (this.CH_Obj.Text == "")
          {
            int num = (int) MessageBox.Show("Input Channel");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            return;
          }
          int int32_3 = Convert.ToInt32(this.CH_Obj.Text.Trim());
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer3 = new byte[6]
          {
            (byte) 170,
            (byte) 0,
            (byte) 3,
            (byte) 1,
            (byte) int32_3,
            byte.MaxValue
          };
          this.sPort.Write(buffer3, 0, buffer3.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer3) + "\n");
          Form1.Delay(300);
          this.sPort.Write(buffer3, 0, buffer3.Length);
          this.History_Obj.SelectionColor = Color.Blue;
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer3) + "\n");
          Form1.Delay(300);
          this.sPort.Write(buffer3, 0, buffer3.Length);
          this.History_Obj.SelectionColor = Color.Blue;
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer3) + "\n");
          Form1.Delay(300);
          this.sPort.Write(buffer3, 0, buffer3.Length);
          this.History_Obj.SelectionColor = Color.Blue;
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer3) + "\n");
          Form1.Delay(300);
          this.sPort.Write(buffer3, 0, buffer3.Length);
          this.History_Obj.SelectionColor = Color.Blue;
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer3) + "\n");
          Form1.Delay(300);
          if (this.Delay_Obj.Text == "")
          {
            int num = (int) MessageBox.Show("Input Delay");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            return;
          }
          string hex = this.DecToHex(this.Delay_Obj.Text.Trim());
          byte[] numArray = new byte[hex.Length];
          int length = numArray.Length;
          for (int index = 0; index < length; ++index)
            numArray[index] = Convert.ToByte(hex.Substring(index * 2), 16);
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer4 = new byte[6]
          {
            (byte) 170,
            (byte) 0,
            (byte) 163,
            (byte) 1,
            numArray[0],
            byte.MaxValue
          };
          this.sPort.Write(buffer4, 0, buffer4.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer4) + "\n");
          this.History_Obj.ScrollToCaret();
          Form1.Delay(300);
          if (this.MAP_MAC.Text == "")
          {
            int num = (int) MessageBox.Show("Input MAC");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            return;
          }
          if (this.MAP_MAC.Text.Length != 4)
          {
            int num = (int) MessageBox.Show("Input Correct MAC");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            return;
          }
          int int32_4 = Convert.ToInt32(this.HexToDec(this.MAP_MAC.Text.Substring(0, 2)));
          int int32_5 = Convert.ToInt32(this.HexToDec(this.MAP_MAC.Text.Substring(2, 2)));
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer5 = new byte[7]
          {
            (byte) 170,
            (byte) 0,
            (byte) 16,
            (byte) 2,
            (byte) int32_4,
            (byte) int32_5,
            byte.MaxValue
          };
          this.sPort.Write(buffer5, 0, buffer5.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer5) + "\n");
          this.History_Obj.ScrollToCaret();
          Form1.Delay(300);
          if (this.MAP_CH.Text == "")
          {
            int num = (int) MessageBox.Show("Input Channel");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            return;
          }
          int int32_6 = Convert.ToInt32(this.MAP_CH.Text.Trim());
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer6 = new byte[6]
          {
            (byte) 170,
            (byte) 0,
            (byte) 18,
            (byte) 1,
            (byte) int32_6,
            byte.MaxValue
          };
          this.sPort.Write(buffer6, 0, buffer6.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer6) + "\n");
          this.History_Obj.ScrollToCaret();
          Form1.Delay(300);
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer7 = new byte[5]
          {
            (byte) 170,
            (byte) 0,
            (byte) 165,
            (byte) 0,
            byte.MaxValue
          };
          this.sPort.Write(buffer7, 0, buffer7.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer7) + "\n");
          this.History_Obj.ScrollToCaret();
          Form1.Delay(300);
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer8 = new byte[5]
          {
            (byte) 170,
            (byte) 0,
            (byte) 166,
            (byte) 0,
            byte.MaxValue
          };
          this.sPort.Write(buffer8, 0, buffer8.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer1) + "\n");
          this.History_Obj.ScrollToCaret();
        }
        if (this.radioButton1.Checked)
        {
          if (this.Type_Obj.Text == "")
          {
            int num = (int) MessageBox.Show("Select Type");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            return;
          }
          this.History_Obj.Clear();
          if (this.Type_Obj.Text == "READ MAC")
          {
            this.GO_Obj.Enabled = false;
            this.History_Obj.SelectionColor = Color.Blue;
            byte[] buffer = new byte[5]
            {
              (byte) 170,
              (byte) 0,
              (byte) 5,
              (byte) 0,
              byte.MaxValue
            };
            this.sPort.Write(buffer, 0, buffer.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
            this.History_Obj.ScrollToCaret();
          }
          if (this.Type_Obj.Text == "READ Channel")
          {
            this.GO_Obj.Enabled = false;
            this.History_Obj.SelectionColor = Color.Blue;
            byte[] buffer = new byte[5]
            {
              (byte) 170,
              (byte) 0,
              (byte) 6,
              (byte) 0,
              byte.MaxValue
            };
            this.sPort.Write(buffer, 0, buffer.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
            this.History_Obj.ScrollToCaret();
          }
          if (this.Type_Obj.Text == "READ MAC+Channel")
          {
            this.GO_Obj.Enabled = false;
            this.Type_Obj.Text = "READ MAC";
            this.History_Obj.SelectionColor = Color.Blue;
            byte[] buffer9 = new byte[5]
            {
              (byte) 170,
              (byte) 0,
              (byte) 5,
              (byte) 0,
              byte.MaxValue
            };
            this.sPort.Write(buffer9, 0, buffer9.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer9) + "\n");
            this.History_Obj.ScrollToCaret();
            Form1.Delay(500);
            this.Type_Obj.Text = "READ Channel";
            this.History_Obj.SelectionColor = Color.Blue;
            byte[] buffer10 = new byte[5]
            {
              (byte) 170,
              (byte) 0,
              (byte) 6,
              (byte) 0,
              byte.MaxValue
            };
            this.sPort.Write(buffer10, 0, buffer10.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer10) + "\n");
            this.History_Obj.ScrollToCaret();
            Form1.Delay(500);
            this.Type_Obj.Text = "READ MAC+Channel";
          }
          if (this.Type_Obj.Text == "READ F/W Info")
          {
            this.GO_Obj.Enabled = false;
            this.Type_Obj.Text = "READ F/W info";
            this.History_Obj.SelectionColor = Color.Blue;
            byte[] buffer = new byte[5]
            {
              (byte) 170,
              (byte) 0,
              (byte) 1,
              (byte) 0,
              byte.MaxValue
            };
            this.sPort.Write(buffer, 0, buffer.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
            this.History_Obj.ScrollToCaret();
            Form1.Delay(300);
            this.Type_Obj.Text = "READ F/W Info";
          }
          if (this.Type_Obj.Text == "Write MAC")
          {
            this.button1.Enabled = false;
            if (this.MAC_Obj.Text == "")
            {
              int num = (int) MessageBox.Show("Input MAC");
              this.GO_Obj.Enabled = true;
              this.Type_Obj.Enabled = true;
              return;
            }
            if (this.MAC_Obj.Text.Length != 4)
            {
              int num = (int) MessageBox.Show("Input Correct MAC");
              this.GO_Obj.Enabled = true;
              this.Type_Obj.Enabled = true;
              return;
            }
            int int32_7 = Convert.ToInt32(this.HexToDec(this.MAC_Obj.Text.Substring(0, 2)));
            int int32_8 = Convert.ToInt32(this.HexToDec(this.MAC_Obj.Text.Substring(2, 2)));
            this.History_Obj.SelectionColor = Color.Blue;
            byte[] buffer = new byte[7]
            {
              (byte) 170,
              (byte) 0,
              (byte) 2,
              (byte) 2,
              (byte) int32_7,
              (byte) int32_8,
              byte.MaxValue
            };
            this.sPort.Write(buffer, 0, buffer.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
            Form1.Delay(200);
            this.sPort.Write(buffer, 0, buffer.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
            Form1.Delay(200);
            this.sPort.Write(buffer, 0, buffer.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
            Form1.Delay(200);
            this.sPort.Write(buffer, 0, buffer.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
            Form1.Delay(200);
            this.sPort.Write(buffer, 0, buffer.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
            this.History_Obj.ScrollToCaret();
          }
          if (this.Type_Obj.Text == "Write Channel")
          {
            this.button1.Enabled = false;
            if (this.CH_Obj.Text == "")
            {
              int num = (int) MessageBox.Show("Input Channel");
              this.GO_Obj.Enabled = true;
              this.Type_Obj.Enabled = true;
              return;
            }
            int int32 = Convert.ToInt32(this.CH_Obj.Text.Trim());
            this.History_Obj.SelectionColor = Color.Blue;
            byte[] buffer = new byte[6]
            {
              (byte) 170,
              (byte) 0,
              (byte) 3,
              (byte) 1,
              (byte) int32,
              byte.MaxValue
            };
            this.sPort.Write(buffer, 0, buffer.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
            Form1.Delay(200);
            this.sPort.Write(buffer, 0, buffer.Length);
            this.History_Obj.SelectionColor = Color.Blue;
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
            Form1.Delay(200);
            this.sPort.Write(buffer, 0, buffer.Length);
            this.History_Obj.SelectionColor = Color.Blue;
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
            Form1.Delay(200);
            this.sPort.Write(buffer, 0, buffer.Length);
            this.History_Obj.SelectionColor = Color.Blue;
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
            Form1.Delay(200);
            this.sPort.Write(buffer, 0, buffer.Length);
            this.History_Obj.SelectionColor = Color.Blue;
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
            this.History_Obj.ScrollToCaret();
          }
          if (this.Type_Obj.Text == "Write MAC+Channel")
          {
            this.button1.Enabled = false;
            this.Type_Obj.Text = "Write MAC";
            this.History_Obj.SelectionColor = Color.Blue;
            if (this.MAC_Obj.Text == "")
            {
              int num = (int) MessageBox.Show("Input MAC");
              this.GO_Obj.Enabled = true;
              this.Type_Obj.Enabled = true;
              this.Type_Obj.Text = "Write MAC+Channel";
              return;
            }
            if (this.MAC_Obj.Text.Length != 4)
            {
              int num = (int) MessageBox.Show("Input Correct MAC");
              this.GO_Obj.Enabled = true;
              this.Type_Obj.Enabled = true;
              this.Type_Obj.Text = "Write MAC+Channel";
              return;
            }
            int int32_9 = Convert.ToInt32(this.HexToDec(this.MAC_Obj.Text.Substring(0, 2)));
            int int32_10 = Convert.ToInt32(this.HexToDec(this.MAC_Obj.Text.Substring(2, 2)));
            this.History_Obj.SelectionColor = Color.Blue;
            byte[] buffer11 = new byte[7]
            {
              (byte) 170,
              (byte) 0,
              (byte) 2,
              (byte) 2,
              (byte) int32_9,
              (byte) int32_10,
              byte.MaxValue
            };
            this.sPort.Write(buffer11, 0, buffer11.Length);
            this.History_Obj.SelectionColor = Color.Blue;
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer11) + "\n");
            Form1.Delay(200);
            this.sPort.Write(buffer11, 0, buffer11.Length);
            this.History_Obj.SelectionColor = Color.Blue;
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer11) + "\n");
            Form1.Delay(200);
            this.sPort.Write(buffer11, 0, buffer11.Length);
            this.History_Obj.SelectionColor = Color.Blue;
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer11) + "\n");
            Form1.Delay(200);
            this.sPort.Write(buffer11, 0, buffer11.Length);
            this.History_Obj.SelectionColor = Color.Blue;
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer11) + "\n");
            Form1.Delay(200);
            this.sPort.Write(buffer11, 0, buffer11.Length);
            this.History_Obj.SelectionColor = Color.Blue;
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer11) + "\n");
            this.History_Obj.ScrollToCaret();
            Form1.Delay(500);
            this.Type_Obj.Text = "Write Channel";
            if (this.CH_Obj.Text == "")
            {
              int num = (int) MessageBox.Show("Input Channel");
              this.GO_Obj.Enabled = true;
              this.Type_Obj.Enabled = true;
              this.Type_Obj.Text = "Write MAC+Channel";
              return;
            }
            int int32_11 = Convert.ToInt32(this.CH_Obj.Text.Trim());
            this.History_Obj.SelectionColor = Color.Blue;
            byte[] buffer12 = new byte[6]
            {
              (byte) 170,
              (byte) 0,
              (byte) 3,
              (byte) 1,
              (byte) int32_11,
              byte.MaxValue
            };
            this.sPort.Write(buffer12, 0, buffer12.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer12) + "\n");
            Form1.Delay(200);
            this.sPort.Write(buffer12, 0, buffer12.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer12) + "\n");
            Form1.Delay(200);
            this.sPort.Write(buffer12, 0, buffer12.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer12) + "\n");
            Form1.Delay(200);
            this.sPort.Write(buffer12, 0, buffer12.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer12) + "\n");
            Form1.Delay(200);
            this.sPort.Write(buffer12, 0, buffer12.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer12) + "\n");
            this.History_Obj.ScrollToCaret();
            Form1.Delay(500);
            this.Type_Obj.Text = "Write MAC+Channel";
          }
          if (this.Type_Obj.Text == "Write Mapping MAC")
          {
            this.button1.Enabled = false;
            if (this.MAP_MAC.Text == "")
            {
              int num = (int) MessageBox.Show("Input MAC");
              this.GO_Obj.Enabled = true;
              this.Type_Obj.Enabled = true;
              return;
            }
            if (this.MAP_MAC.Text.Length != 4)
            {
              int num = (int) MessageBox.Show("Input Correct MAC");
              this.GO_Obj.Enabled = true;
              this.Type_Obj.Enabled = true;
              return;
            }
            int int32_12 = Convert.ToInt32(this.HexToDec(this.MAP_MAC.Text.Substring(0, 2)));
            int int32_13 = Convert.ToInt32(this.HexToDec(this.MAP_MAC.Text.Substring(2, 2)));
            this.History_Obj.SelectionColor = Color.Blue;
            byte[] buffer = new byte[7]
            {
              (byte) 170,
              (byte) 0,
              (byte) 16,
              (byte) 2,
              (byte) int32_12,
              (byte) int32_13,
              byte.MaxValue
            };
            this.sPort.Write(buffer, 0, buffer.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
            this.History_Obj.ScrollToCaret();
          }
          if (this.Type_Obj.Text == "Write Mapping Channel")
          {
            this.button1.Enabled = false;
            if (this.MAP_CH.Text == "")
            {
              int num = (int) MessageBox.Show("Input Channel");
              this.GO_Obj.Enabled = true;
              this.Type_Obj.Enabled = true;
              return;
            }
            int int32 = Convert.ToInt32(this.MAP_CH.Text.Trim());
            this.History_Obj.SelectionColor = Color.Blue;
            byte[] buffer = new byte[6]
            {
              (byte) 170,
              (byte) 0,
              (byte) 18,
              (byte) 1,
              (byte) int32,
              byte.MaxValue
            };
            this.sPort.Write(buffer, 0, buffer.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
            this.History_Obj.ScrollToCaret();
          }
          if (this.Type_Obj.Text == "Write Mapping MAC+Channel")
          {
            this.button1.Enabled = false;
            this.Type_Obj.Text = "Write Mapping MAC";
            if (this.MAP_MAC.Text == "")
            {
              int num = (int) MessageBox.Show("Input MAC");
              this.GO_Obj.Enabled = true;
              this.Type_Obj.Enabled = true;
              this.Type_Obj.Text = "Write Mapping MAC+Channel";
              return;
            }
            if (this.MAP_MAC.Text.Length != 4)
            {
              int num = (int) MessageBox.Show("Input Correct MAC");
              this.GO_Obj.Enabled = true;
              this.Type_Obj.Enabled = true;
              return;
            }
            int int32_14 = Convert.ToInt32(this.HexToDec(this.MAP_MAC.Text.Substring(0, 2)));
            int int32_15 = Convert.ToInt32(this.HexToDec(this.MAP_MAC.Text.Substring(2, 2)));
            this.History_Obj.SelectionColor = Color.Blue;
            byte[] buffer13 = new byte[7]
            {
              (byte) 170,
              (byte) 0,
              (byte) 16,
              (byte) 2,
              (byte) int32_14,
              (byte) int32_15,
              byte.MaxValue
            };
            this.sPort.Write(buffer13, 0, buffer13.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer13) + "\n");
            this.History_Obj.ScrollToCaret();
            Form1.Delay(300);
            this.Type_Obj.Text = "Write Mapping Channel";
            if (this.MAP_CH.Text == "")
            {
              int num = (int) MessageBox.Show("Input Channel");
              this.GO_Obj.Enabled = true;
              this.Type_Obj.Enabled = true;
              return;
            }
            int int32_16 = Convert.ToInt32(this.MAP_CH.Text.Trim());
            this.History_Obj.SelectionColor = Color.Blue;
            byte[] buffer14 = new byte[6]
            {
              (byte) 170,
              (byte) 0,
              (byte) 18,
              (byte) 1,
              (byte) int32_16,
              byte.MaxValue
            };
            this.sPort.Write(buffer14, 0, buffer14.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer14) + "\n");
            this.History_Obj.ScrollToCaret();
            Form1.Delay(300);
            this.Type_Obj.Text = "Write Mapping MAC+Channel";
          }
          if (this.Type_Obj.Text == "READ Mapping MAC+Channel")
          {
            this.GO_Obj.Enabled = false;
            this.Type_Obj.Text = "READ MAC";
            this.History_Obj.SelectionColor = Color.Blue;
            byte[] buffer15 = new byte[5]
            {
              (byte) 170,
              (byte) 0,
              (byte) 17,
              (byte) 0,
              byte.MaxValue
            };
            this.sPort.Write(buffer15, 0, buffer15.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer15) + "\n");
            this.History_Obj.ScrollToCaret();
            Form1.Delay(300);
            this.Type_Obj.Text = "READ Channel";
            this.History_Obj.SelectionColor = Color.Blue;
            byte[] buffer16 = new byte[5]
            {
              (byte) 170,
              (byte) 0,
              (byte) 19,
              (byte) 0,
              byte.MaxValue
            };
            this.sPort.Write(buffer16, 0, buffer16.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer16) + "\n");
            this.History_Obj.ScrollToCaret();
            Form1.Delay(300);
            this.Type_Obj.Text = "READ MAC+Channel";
          }
          if (this.Type_Obj.Text == "Write Device Info Delay")
          {
            this.button1.Enabled = false;
            if (this.Delay_Obj.Text == "")
            {
              int num = (int) MessageBox.Show("Input Delay");
              this.GO_Obj.Enabled = true;
              this.Type_Obj.Enabled = true;
              return;
            }
            string hex = this.DecToHex(this.Delay_Obj.Text.Trim());
            byte[] numArray = new byte[hex.Length];
            int length = numArray.Length;
            for (int index = 0; index < length; ++index)
              numArray[index] = Convert.ToByte(hex.Substring(index * 2), 16);
            this.History_Obj.SelectionColor = Color.Blue;
            byte[] buffer = new byte[6]
            {
              (byte) 170,
              (byte) 0,
              (byte) 163,
              (byte) 1,
              numArray[0],
              byte.MaxValue
            };
            this.sPort.Write(buffer, 0, buffer.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
            this.History_Obj.ScrollToCaret();
          }
          if (this.Type_Obj.Text == "READ Device Info Delay")
          {
            this.GO_Obj.Enabled = false;
            this.History_Obj.SelectionColor = Color.Blue;
            byte[] buffer = new byte[5]
            {
              (byte) 170,
              (byte) 0,
              (byte) 164,
              (byte) 0,
              byte.MaxValue
            };
            this.sPort.Write(buffer, 0, buffer.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
            this.History_Obj.ScrollToCaret();
          }
          if (this.Type_Obj.Text == "Copy to default area")
          {
            this.GO_Obj.Enabled = false;
            this.History_Obj.SelectionColor = Color.Blue;
            byte[] buffer = new byte[5]
            {
              (byte) 170,
              (byte) 0,
              (byte) 165,
              (byte) 0,
              byte.MaxValue
            };
            this.sPort.Write(buffer, 0, buffer.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
            this.History_Obj.ScrollToCaret();
          }
          if (this.Type_Obj.Text == "Default area data comparison")
          {
            this.button1.Enabled = false;
            this.History_Obj.SelectionColor = Color.Blue;
            byte[] buffer = new byte[5]
            {
              (byte) 170,
              (byte) 0,
              (byte) 166,
              (byte) 0,
              byte.MaxValue
            };
            this.sPort.Write(buffer, 0, buffer.Length);
            this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
            this.History_Obj.ScrollToCaret();
          }
        }
        this.GO_Obj.Enabled = true;
        this.button1.Enabled = true;
        this.Type_Obj.Enabled = true;
      }
      catch
      {
        this.GO_Obj.Enabled = true;
        this.Type_Obj.Enabled = true;
        this.button1.Enabled = true;
        this.History_Obj.AppendText("[Error] Check Input Data \n");
      }
    }

    private void Form1_KeyDown(object sender, KeyEventArgs e)
    {
      if (e.KeyCode == Keys.F2)
        this.GO_Obj.PerformClick();
      if (e.KeyCode != Keys.F1)
        return;
      this.Connect_btn.PerformClick();
    }

    private void MACResult_Obj_Click(object sender, EventArgs e)
    {
    }

    private void button2_Click(object sender, EventArgs e)
    {
      SaveFileDialog saveFileDialog = new SaveFileDialog();
      saveFileDialog.Filter = "txt files (*.txt)|*.txt|All files (*.*)|*.*";
      saveFileDialog.Title = "Save a Text File";
      saveFileDialog.InitialDirectory = Application.StartupPath;
      if (saveFileDialog.ShowDialog() != DialogResult.OK)
        return;
      File.WriteAllText(saveFileDialog.FileName, "This is your text");
    }

    private void button1_Click(object sender, EventArgs e)
    {
      this.History_Obj.SelectionColor = Color.Blue;
      byte[] buffer1 = new byte[5]
      {
        (byte) 170,
        (byte) 0,
        (byte) 1,
        (byte) 0,
        byte.MaxValue
      };
      this.sPort.Write(buffer1, 0, buffer1.Length);
      this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer1) + "\n");
      Form1.Delay(300);
      this.History_Obj.SelectionColor = Color.Blue;
      byte[] buffer2 = new byte[5]
      {
        (byte) 170,
        (byte) 0,
        (byte) 5,
        (byte) 0,
        byte.MaxValue
      };
      this.sPort.Write(buffer2, 0, buffer2.Length);
      this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer1) + "\n");
      Form1.Delay(300);
      this.History_Obj.SelectionColor = Color.Blue;
      byte[] buffer3 = new byte[5]
      {
        (byte) 170,
        (byte) 0,
        (byte) 6,
        (byte) 0,
        byte.MaxValue
      };
      this.sPort.Write(buffer3, 0, buffer3.Length);
      this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer1) + "\n");
      Form1.Delay(300);
      this.History_Obj.SelectionColor = Color.Blue;
      byte[] buffer4 = new byte[5]
      {
        (byte) 170,
        (byte) 0,
        (byte) 164,
        (byte) 0,
        byte.MaxValue
      };
      this.sPort.Write(buffer4, 0, buffer4.Length);
      this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer1) + "\n");
      Form1.Delay(300);
      this.History_Obj.SelectionColor = Color.Blue;
      byte[] buffer5 = new byte[5]
      {
        (byte) 170,
        (byte) 0,
        (byte) 17,
        (byte) 0,
        byte.MaxValue
      };
      this.sPort.Write(buffer5, 0, buffer5.Length);
      this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer1) + "\n");
      Form1.Delay(300);
      this.History_Obj.SelectionColor = Color.Blue;
      byte[] buffer6 = new byte[5]
      {
        (byte) 170,
        (byte) 0,
        (byte) 19,
        (byte) 0,
        byte.MaxValue
      };
      this.sPort.Write(buffer6, 0, buffer6.Length);
      this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer2) + "\n");
      Form1.Delay(300);
      this.History_Obj.SelectionColor = Color.Blue;
      byte[] buffer7 = new byte[5]
      {
        (byte) 170,
        (byte) 0,
        (byte) 166,
        (byte) 0,
        byte.MaxValue
      };
      this.sPort.Write(buffer7, 0, buffer7.Length);
      this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer1) + "\n");
      this.History_Obj.ScrollToCaret();
    }

    private void button2_Click_1(object sender, EventArgs e)
    {
      if (this.MAC_Obj.Text.Length != 4)
      {
        int num = (int) MessageBox.Show("Input Correct MAC");
        this.GO_Obj.Enabled = true;
        this.Type_Obj.Enabled = true;
        this.Type_Obj.Text = "Write MAC+Channel";
      }
      else if (string.IsNullOrWhiteSpace(this.MAC_Obj.Text) || string.IsNullOrWhiteSpace(this.CH_Obj.Text) || string.IsNullOrWhiteSpace(this.Delay_Obj.Text) || string.IsNullOrWhiteSpace(this.MAP_MAC.Text) || string.IsNullOrWhiteSpace(this.MAP_CH.Text))
      {
        int num1 = (int) MessageBox.Show("Please fill in all fields.");
      }
      else if (this.CH_Obj.Text != this.MAP_CH.Text)
      {
        int num2 = (int) MessageBox.Show("The ZIGBEE channel and the MAPPING channel must be the same.");
      }
      else
      {
        int result;
        if (int.TryParse(new string(this.MAP_MAC.Text.Where<char>(new Func<char, bool>(char.IsDigit)).ToArray<char>()), out result) && result % 2 == 0)
        {
          int num3 = (int) MessageBox.Show("MAPPING RX MAC must be odd.");
        }
        else
        {
          SaveFileDialog saveFileDialog = new SaveFileDialog();
          saveFileDialog.InitialDirectory = Path.GetDirectoryName(Application.ExecutablePath);
          saveFileDialog.Filter = "txt files (*.txt)|*.txt|All files (*.*)|*.*";
          if (saveFileDialog.ShowDialog() != DialogResult.OK)
            return;
          string fileName = saveFileDialog.FileName;
          using (StreamWriter text = File.CreateText(fileName))
          {
            text.WriteLine(this.MAC_Obj.Text);
            text.WriteLine(this.CH_Obj.Text);
            text.WriteLine(this.Delay_Obj.Text);
            text.WriteLine(this.MAP_MAC.Text);
            text.WriteLine(this.MAP_CH.Text);
          }
          this.UpdateComboBox(this.comboBox1, Path.GetFileName(fileName));
        }
      }
    }

    public void UpdateComboBox(ComboBox comboBox1, string selectedFile = null)
    {
      comboBox1.Items.Clear();
      foreach (string file in Directory.GetFiles(Path.GetDirectoryName(Application.ExecutablePath), "*.txt"))
        comboBox1.Items.Add((object) Path.GetFileName(file));
      if (selectedFile == null)
        return;
      comboBox1.SelectedItem = (object) selectedFile;
    }

    private void button1_Click_1(object sender, EventArgs e)
    {
      if (this.radioButton2.Checked)
      {
        this.GO_Obj.Enabled = false;
        this.FW_Ver.Text = "";
        this.MAC_Obj.Text = "";
        this.CH_Obj.Text = "";
        this.Delay_Obj.Text = "";
        this.MAP_MAC.Text = "";
        this.MAP_CH.Text = "";
        this.DATA_COMPARE.Text = "";
        this.History_Obj.SelectionColor = Color.Blue;
        byte[] buffer1 = new byte[5]
        {
          (byte) 170,
          (byte) 0,
          (byte) 1,
          (byte) 0,
          byte.MaxValue
        };
        this.sPort.Write(buffer1, 0, buffer1.Length);
        this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer1) + "\n");
        Form1.Delay(300);
        this.History_Obj.SelectionColor = Color.Blue;
        byte[] buffer2 = new byte[5]
        {
          (byte) 170,
          (byte) 0,
          (byte) 5,
          (byte) 0,
          byte.MaxValue
        };
        this.sPort.Write(buffer2, 0, buffer2.Length);
        this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer1) + "\n");
        Form1.Delay(300);
        this.History_Obj.SelectionColor = Color.Blue;
        byte[] buffer3 = new byte[5]
        {
          (byte) 170,
          (byte) 0,
          (byte) 6,
          (byte) 0,
          byte.MaxValue
        };
        this.sPort.Write(buffer3, 0, buffer3.Length);
        this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer1) + "\n");
        Form1.Delay(300);
        this.History_Obj.SelectionColor = Color.Blue;
        byte[] buffer4 = new byte[5]
        {
          (byte) 170,
          (byte) 0,
          (byte) 164,
          (byte) 0,
          byte.MaxValue
        };
        this.sPort.Write(buffer4, 0, buffer4.Length);
        this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer1) + "\n");
        Form1.Delay(300);
        this.History_Obj.SelectionColor = Color.Blue;
        byte[] buffer5 = new byte[5]
        {
          (byte) 170,
          (byte) 0,
          (byte) 17,
          (byte) 0,
          byte.MaxValue
        };
        this.sPort.Write(buffer5, 0, buffer5.Length);
        this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer1) + "\n");
        Form1.Delay(300);
        this.History_Obj.SelectionColor = Color.Blue;
        byte[] buffer6 = new byte[5]
        {
          (byte) 170,
          (byte) 0,
          (byte) 19,
          (byte) 0,
          byte.MaxValue
        };
        this.sPort.Write(buffer6, 0, buffer6.Length);
        this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer2) + "\n");
        Form1.Delay(300);
        this.History_Obj.SelectionColor = Color.Blue;
        byte[] buffer7 = new byte[5]
        {
          (byte) 170,
          (byte) 0,
          (byte) 166,
          (byte) 0,
          byte.MaxValue
        };
        this.sPort.Write(buffer7, 0, buffer7.Length);
        this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer1) + "\n");
        this.History_Obj.ScrollToCaret();
        this.GO_Obj.Enabled = true;
      }
      if (!this.radioButton1.Checked)
        return;
      this.FW_Ver.Text = "";
      this.MAC_Obj.Text = "";
      this.CH_Obj.Text = "";
      this.Delay_Obj.Text = "";
      this.MAP_MAC.Text = "";
      this.MAP_CH.Text = "";
      this.DATA_COMPARE.Text = "";
      if (this.Type_Obj.Text == "")
      {
        int num = (int) MessageBox.Show("Select Type");
        this.GO_Obj.Enabled = true;
        this.Type_Obj.Enabled = true;
      }
      else
      {
        this.History_Obj.Clear();
        if (this.Type_Obj.Text == "READ MAC")
        {
          this.GO_Obj.Enabled = false;
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer = new byte[5]
          {
            (byte) 170,
            (byte) 0,
            (byte) 5,
            (byte) 0,
            byte.MaxValue
          };
          this.sPort.Write(buffer, 0, buffer.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
          this.History_Obj.ScrollToCaret();
        }
        if (this.Type_Obj.Text == "READ Channel")
        {
          this.GO_Obj.Enabled = false;
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer = new byte[5]
          {
            (byte) 170,
            (byte) 0,
            (byte) 6,
            (byte) 0,
            byte.MaxValue
          };
          this.sPort.Write(buffer, 0, buffer.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
          this.History_Obj.ScrollToCaret();
        }
        if (this.Type_Obj.Text == "READ MAC+Channel")
        {
          this.GO_Obj.Enabled = false;
          this.Type_Obj.Text = "READ MAC";
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer8 = new byte[5]
          {
            (byte) 170,
            (byte) 0,
            (byte) 5,
            (byte) 0,
            byte.MaxValue
          };
          this.sPort.Write(buffer8, 0, buffer8.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer8) + "\n");
          this.History_Obj.ScrollToCaret();
          Form1.Delay(500);
          this.Type_Obj.Text = "READ Channel";
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer9 = new byte[5]
          {
            (byte) 170,
            (byte) 0,
            (byte) 6,
            (byte) 0,
            byte.MaxValue
          };
          this.sPort.Write(buffer9, 0, buffer9.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer9) + "\n");
          this.History_Obj.ScrollToCaret();
          Form1.Delay(500);
          this.Type_Obj.Text = "READ MAC+Channel";
        }
        if (this.Type_Obj.Text == "READ F/W Info")
        {
          this.GO_Obj.Enabled = false;
          this.Type_Obj.Text = "READ F/W info";
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer = new byte[5]
          {
            (byte) 170,
            (byte) 0,
            (byte) 1,
            (byte) 0,
            byte.MaxValue
          };
          this.sPort.Write(buffer, 0, buffer.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
          this.History_Obj.ScrollToCaret();
          Form1.Delay(300);
          this.Type_Obj.Text = "READ F/W Info";
        }
        if (this.Type_Obj.Text == "Write MAC")
        {
          this.button1.Enabled = false;
          if (this.MAC_Obj.Text == "")
          {
            int num = (int) MessageBox.Show("Input MAC");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            return;
          }
          if (this.MAC_Obj.Text.Length != 4)
          {
            int num = (int) MessageBox.Show("Input Correct MAC");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            return;
          }
          int int32_1 = Convert.ToInt32(this.HexToDec(this.MAC_Obj.Text.Substring(0, 2)));
          int int32_2 = Convert.ToInt32(this.HexToDec(this.MAC_Obj.Text.Substring(2, 2)));
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer = new byte[7]
          {
            (byte) 170,
            (byte) 0,
            (byte) 2,
            (byte) 2,
            (byte) int32_1,
            (byte) int32_2,
            byte.MaxValue
          };
          this.sPort.Write(buffer, 0, buffer.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
          Form1.Delay(200);
          this.sPort.Write(buffer, 0, buffer.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
          Form1.Delay(200);
          this.sPort.Write(buffer, 0, buffer.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
          Form1.Delay(200);
          this.sPort.Write(buffer, 0, buffer.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
          Form1.Delay(200);
          this.sPort.Write(buffer, 0, buffer.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
          this.History_Obj.ScrollToCaret();
        }
        if (this.Type_Obj.Text == "Write Channel")
        {
          this.button1.Enabled = false;
          if (this.CH_Obj.Text == "")
          {
            int num = (int) MessageBox.Show("Input Channel");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            return;
          }
          int int32 = Convert.ToInt32(this.CH_Obj.Text.Trim());
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer = new byte[6]
          {
            (byte) 170,
            (byte) 0,
            (byte) 3,
            (byte) 1,
            (byte) int32,
            byte.MaxValue
          };
          this.sPort.Write(buffer, 0, buffer.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
          Form1.Delay(200);
          this.sPort.Write(buffer, 0, buffer.Length);
          this.History_Obj.SelectionColor = Color.Blue;
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
          Form1.Delay(200);
          this.sPort.Write(buffer, 0, buffer.Length);
          this.History_Obj.SelectionColor = Color.Blue;
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
          Form1.Delay(200);
          this.sPort.Write(buffer, 0, buffer.Length);
          this.History_Obj.SelectionColor = Color.Blue;
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
          Form1.Delay(200);
          this.sPort.Write(buffer, 0, buffer.Length);
          this.History_Obj.SelectionColor = Color.Blue;
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
          this.History_Obj.ScrollToCaret();
        }
        if (this.Type_Obj.Text == "Write MAC+Channel")
        {
          this.button1.Enabled = false;
          this.Type_Obj.Text = "Write MAC";
          this.History_Obj.SelectionColor = Color.Blue;
          if (this.MAC_Obj.Text == "")
          {
            int num = (int) MessageBox.Show("Input MAC");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            this.Type_Obj.Text = "Write MAC+Channel";
            return;
          }
          if (this.MAC_Obj.Text.Length != 4)
          {
            int num = (int) MessageBox.Show("Input Correct MAC");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            this.Type_Obj.Text = "Write MAC+Channel";
            return;
          }
          int int32_3 = Convert.ToInt32(this.HexToDec(this.MAC_Obj.Text.Substring(0, 2)));
          int int32_4 = Convert.ToInt32(this.HexToDec(this.MAC_Obj.Text.Substring(2, 2)));
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer10 = new byte[7]
          {
            (byte) 170,
            (byte) 0,
            (byte) 2,
            (byte) 2,
            (byte) int32_3,
            (byte) int32_4,
            byte.MaxValue
          };
          this.sPort.Write(buffer10, 0, buffer10.Length);
          this.History_Obj.SelectionColor = Color.Blue;
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer10) + "\n");
          Form1.Delay(200);
          this.sPort.Write(buffer10, 0, buffer10.Length);
          this.History_Obj.SelectionColor = Color.Blue;
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer10) + "\n");
          Form1.Delay(200);
          this.sPort.Write(buffer10, 0, buffer10.Length);
          this.History_Obj.SelectionColor = Color.Blue;
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer10) + "\n");
          Form1.Delay(200);
          this.sPort.Write(buffer10, 0, buffer10.Length);
          this.History_Obj.SelectionColor = Color.Blue;
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer10) + "\n");
          Form1.Delay(200);
          this.sPort.Write(buffer10, 0, buffer10.Length);
          this.History_Obj.SelectionColor = Color.Blue;
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer10) + "\n");
          this.History_Obj.ScrollToCaret();
          Form1.Delay(500);
          this.Type_Obj.Text = "Write Channel";
          if (this.CH_Obj.Text == "")
          {
            int num = (int) MessageBox.Show("Input Channel");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            this.Type_Obj.Text = "Write MAC+Channel";
            return;
          }
          int int32_5 = Convert.ToInt32(this.CH_Obj.Text.Trim());
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer11 = new byte[6]
          {
            (byte) 170,
            (byte) 0,
            (byte) 3,
            (byte) 1,
            (byte) int32_5,
            byte.MaxValue
          };
          this.sPort.Write(buffer11, 0, buffer11.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer11) + "\n");
          Form1.Delay(200);
          this.sPort.Write(buffer11, 0, buffer11.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer11) + "\n");
          Form1.Delay(200);
          this.sPort.Write(buffer11, 0, buffer11.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer11) + "\n");
          Form1.Delay(200);
          this.sPort.Write(buffer11, 0, buffer11.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer11) + "\n");
          Form1.Delay(200);
          this.sPort.Write(buffer11, 0, buffer11.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer11) + "\n");
          this.History_Obj.ScrollToCaret();
          Form1.Delay(500);
          this.Type_Obj.Text = "Write MAC+Channel";
        }
        if (this.Type_Obj.Text == "Write Mapping MAC")
        {
          this.button1.Enabled = false;
          if (this.MAP_MAC.Text == "")
          {
            int num = (int) MessageBox.Show("Input MAC");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            return;
          }
          if (this.MAP_MAC.Text.Length != 4)
          {
            int num = (int) MessageBox.Show("Input Correct MAC");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            return;
          }
          int int32_6 = Convert.ToInt32(this.HexToDec(this.MAP_MAC.Text.Substring(0, 2)));
          int int32_7 = Convert.ToInt32(this.HexToDec(this.MAP_MAC.Text.Substring(2, 2)));
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer = new byte[7]
          {
            (byte) 170,
            (byte) 0,
            (byte) 16,
            (byte) 2,
            (byte) int32_6,
            (byte) int32_7,
            byte.MaxValue
          };
          this.sPort.Write(buffer, 0, buffer.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
          this.History_Obj.ScrollToCaret();
        }
        if (this.Type_Obj.Text == "Write Mapping Channel")
        {
          this.button1.Enabled = false;
          if (this.MAP_CH.Text == "")
          {
            int num = (int) MessageBox.Show("Input Channel");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            return;
          }
          int int32 = Convert.ToInt32(this.MAP_CH.Text.Trim());
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer = new byte[6]
          {
            (byte) 170,
            (byte) 0,
            (byte) 18,
            (byte) 1,
            (byte) int32,
            byte.MaxValue
          };
          this.sPort.Write(buffer, 0, buffer.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
          this.History_Obj.ScrollToCaret();
        }
        if (this.Type_Obj.Text == "Write Mapping MAC+Channel")
        {
          this.button1.Enabled = false;
          this.Type_Obj.Text = "Write Mapping MAC";
          if (this.MAP_MAC.Text == "")
          {
            int num = (int) MessageBox.Show("Input MAC");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            this.Type_Obj.Text = "Write Mapping MAC+Channel";
            return;
          }
          if (this.MAP_MAC.Text.Length != 4)
          {
            int num = (int) MessageBox.Show("Input Correct MAC");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            return;
          }
          int int32_8 = Convert.ToInt32(this.HexToDec(this.MAP_MAC.Text.Substring(0, 2)));
          int int32_9 = Convert.ToInt32(this.HexToDec(this.MAP_MAC.Text.Substring(2, 2)));
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer12 = new byte[7]
          {
            (byte) 170,
            (byte) 0,
            (byte) 16,
            (byte) 2,
            (byte) int32_8,
            (byte) int32_9,
            byte.MaxValue
          };
          this.sPort.Write(buffer12, 0, buffer12.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer12) + "\n");
          this.History_Obj.ScrollToCaret();
          Form1.Delay(300);
          this.Type_Obj.Text = "Write Mapping Channel";
          if (this.MAP_CH.Text == "")
          {
            int num = (int) MessageBox.Show("Input Channel");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            return;
          }
          int int32_10 = Convert.ToInt32(this.MAP_CH.Text.Trim());
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer13 = new byte[6]
          {
            (byte) 170,
            (byte) 0,
            (byte) 18,
            (byte) 1,
            (byte) int32_10,
            byte.MaxValue
          };
          this.sPort.Write(buffer13, 0, buffer13.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer13) + "\n");
          this.History_Obj.ScrollToCaret();
          Form1.Delay(300);
          this.Type_Obj.Text = "Write Mapping MAC+Channel";
        }
        if (this.Type_Obj.Text == "READ Mapping MAC+Channel")
        {
          this.GO_Obj.Enabled = false;
          this.Type_Obj.Text = "READ MAC";
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer14 = new byte[5]
          {
            (byte) 170,
            (byte) 0,
            (byte) 17,
            (byte) 0,
            byte.MaxValue
          };
          this.sPort.Write(buffer14, 0, buffer14.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer14) + "\n");
          this.History_Obj.ScrollToCaret();
          Form1.Delay(300);
          this.Type_Obj.Text = "READ Channel";
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer15 = new byte[5]
          {
            (byte) 170,
            (byte) 0,
            (byte) 19,
            (byte) 0,
            byte.MaxValue
          };
          this.sPort.Write(buffer15, 0, buffer15.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer15) + "\n");
          this.History_Obj.ScrollToCaret();
          Form1.Delay(300);
          this.Type_Obj.Text = "READ MAC+Channel";
        }
        if (this.Type_Obj.Text == "Write Device Info Delay")
        {
          this.button1.Enabled = false;
          if (this.Delay_Obj.Text == "")
          {
            int num = (int) MessageBox.Show("Input Delay");
            this.GO_Obj.Enabled = true;
            this.Type_Obj.Enabled = true;
            return;
          }
          string hex = this.DecToHex(this.Delay_Obj.Text.Trim());
          byte[] numArray = new byte[hex.Length];
          int length = numArray.Length;
          for (int index = 0; index < length; ++index)
            numArray[index] = Convert.ToByte(hex.Substring(index * 2), 16);
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer = new byte[6]
          {
            (byte) 170,
            (byte) 0,
            (byte) 163,
            (byte) 1,
            numArray[0],
            byte.MaxValue
          };
          this.sPort.Write(buffer, 0, buffer.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
          this.History_Obj.ScrollToCaret();
        }
        if (this.Type_Obj.Text == "READ Device Info Delay")
        {
          this.GO_Obj.Enabled = false;
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer = new byte[5]
          {
            (byte) 170,
            (byte) 0,
            (byte) 164,
            (byte) 0,
            byte.MaxValue
          };
          this.sPort.Write(buffer, 0, buffer.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
          this.History_Obj.ScrollToCaret();
        }
        if (this.Type_Obj.Text == "Copy to default area")
        {
          this.GO_Obj.Enabled = false;
          this.History_Obj.SelectionColor = Color.Blue;
          byte[] buffer = new byte[5]
          {
            (byte) 170,
            (byte) 0,
            (byte) 165,
            (byte) 0,
            byte.MaxValue
          };
          this.sPort.Write(buffer, 0, buffer.Length);
          this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer) + "\n");
          this.History_Obj.ScrollToCaret();
        }
        if (!(this.Type_Obj.Text == "Default area data comparison"))
          return;
        this.button1.Enabled = false;
        this.History_Obj.SelectionColor = Color.Blue;
        byte[] buffer16 = new byte[5]
        {
          (byte) 170,
          (byte) 0,
          (byte) 166,
          (byte) 0,
          byte.MaxValue
        };
        this.sPort.Write(buffer16, 0, buffer16.Length);
        this.History_Obj.AppendText("[Send] " + BitConverter.ToString(buffer16) + "\n");
        this.History_Obj.ScrollToCaret();
      }
    }

    private void button3_Click(object sender, EventArgs e)
    {
      OpenFileDialog openFileDialog = new OpenFileDialog();
      openFileDialog.InitialDirectory = Path.GetDirectoryName(Application.ExecutablePath);
      openFileDialog.Filter = "txt files (*.txt)|*.txt|All files (*.*)|*.*";
      if (openFileDialog.ShowDialog() != DialogResult.OK)
        return;
      string fileName = openFileDialog.FileName;
    }

    private void radioButton1_CheckedChanged(object sender, EventArgs e)
    {
    }

    private void radioButton2_CheckedChanged(object sender, EventArgs e)
    {
    }

    private void Type_Obj_SelectedIndexChanged(object sender, EventArgs e)
    {
      if (this.Type_Obj.SelectedItem.ToString().ToLower().Contains("read") || this.Type_Obj.SelectedItem.ToString().ToLower().Contains("comparison"))
      {
        this.button1.Enabled = true;
        this.GO_Obj.Enabled = false;
      }
      else
      {
        this.button1.Enabled = false;
        this.GO_Obj.Enabled = true;
      }
    }

    private void comboBox1_SelectedIndexChanged_1(object sender, EventArgs e)
    {
    }

    protected override void Dispose(bool disposing)
    {
      if (disposing && this.components != null)
        this.components.Dispose();
      base.Dispose(disposing);
    }

    private void InitializeComponent()
    {
      this.groupBox1 = new GroupBox();
      this.comboBox1 = new ComboBox();
      this.radioButton2 = new RadioButton();
      this.radioButton1 = new RadioButton();
      this.button2 = new Button();
      this.Connect_btn = new Button();
      this.cboPortName = new ComboBox();
      this.Type_Obj = new ComboBox();
      this.label1 = new Label();
      this.label2 = new Label();
      this.label3 = new Label();
      this.MAC_Obj = new TextBox();
      this.CH_Obj = new TextBox();
      this.GO_Obj = new Button();
      this.History_Obj = new RichTextBox();
      this.MACResult_Obj = new Label();
      this.CHResult_Obj = new Label();
      this.label5 = new Label();
      this.Delay_Obj = new TextBox();
      this.button1 = new Button();
      this.MAP_MAC = new TextBox();
      this.label6 = new Label();
      this.label7 = new Label();
      this.MAP_CH = new TextBox();
      this.label8 = new Label();
      this.FW_Ver = new TextBox();
      this.label9 = new Label();
      this.DATA_COMPARE = new TextBox();
      this.groupBox1.SuspendLayout();
      this.SuspendLayout();
      this.groupBox1.Controls.Add((Control) this.comboBox1);
      this.groupBox1.Controls.Add((Control) this.radioButton2);
      this.groupBox1.Controls.Add((Control) this.radioButton1);
      this.groupBox1.Controls.Add((Control) this.button2);
      this.groupBox1.Controls.Add((Control) this.Connect_btn);
      this.groupBox1.Controls.Add((Control) this.cboPortName);
      this.groupBox1.Controls.Add((Control) this.Type_Obj);
      this.groupBox1.Controls.Add((Control) this.label1);
      this.groupBox1.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.groupBox1.Location = new Point(17, 18);
      this.groupBox1.Margin = new Padding(4, 5, 4, 5);
      this.groupBox1.Name = "groupBox1";
      this.groupBox1.Padding = new Padding(4, 5, 4, 5);
      this.groupBox1.Size = new Size(407, 167);
      this.groupBox1.TabIndex = 0;
      this.groupBox1.TabStop = false;
      this.groupBox1.Text = "[Setting]";
      this.comboBox1.FormattingEnabled = true;
      this.comboBox1.Location = new Point(138, 97);
      this.comboBox1.Name = "comboBox1";
      this.comboBox1.Size = new Size(252, 21);
      this.comboBox1.TabIndex = 17;
      this.comboBox1.SelectedIndexChanged += new EventHandler(this.comboBox1_SelectedIndexChanged_1);
      this.radioButton2.AutoSize = true;
      this.radioButton2.Checked = true;
      this.radioButton2.Location = new Point(20, 97);
      this.radioButton2.Name = "radioButton2";
      this.radioButton2.Size = new Size(84, 18);
      this.radioButton2.TabIndex = 4;
      this.radioButton2.TabStop = true;
      this.radioButton2.Text = "Full CMD";
      this.radioButton2.UseVisualStyleBackColor = true;
      this.radioButton2.CheckedChanged += new EventHandler(this.radioButton2_CheckedChanged);
      this.radioButton1.AutoSize = true;
      this.radioButton1.Location = new Point(20, 62);
      this.radioButton1.Name = "radioButton1";
      this.radioButton1.Size = new Size(102, 18);
      this.radioButton1.TabIndex = 3;
      this.radioButton1.TabStop = true;
      this.radioButton1.Text = "Single CMD";
      this.radioButton1.UseVisualStyleBackColor = true;
      this.radioButton1.CheckedChanged += new EventHandler(this.radioButton1_CheckedChanged);
      this.button2.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.button2.Location = new Point(138, 126);
      this.button2.Name = "button2";
      this.button2.Size = new Size(97, 30);
      this.button2.TabIndex = 15;
      this.button2.Text = "FILE SAVE";
      this.button2.UseVisualStyleBackColor = true;
      this.button2.Click += new EventHandler(this.button2_Click_1);
      this.Connect_btn.Location = new Point(213, 19);
      this.Connect_btn.Margin = new Padding(4, 5, 4, 5);
      this.Connect_btn.Name = "Connect_btn";
      this.Connect_btn.Size = new Size(177, 27);
      this.Connect_btn.TabIndex = 2;
      this.Connect_btn.Text = "CONNECT [F1]";
      this.Connect_btn.UseVisualStyleBackColor = true;
      this.Connect_btn.Click += new EventHandler(this.Connect_btn_Click);
      this.cboPortName.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.cboPortName.FormattingEnabled = true;
      this.cboPortName.Location = new Point(88, 22);
      this.cboPortName.Margin = new Padding(4, 5, 4, 5);
      this.cboPortName.Name = "cboPortName";
      this.cboPortName.Size = new Size(113, 21);
      this.cboPortName.TabIndex = 1;
      this.Type_Obj.FormattingEnabled = true;
      this.Type_Obj.Items.AddRange(new object[15]
      {
        (object) "READ MAC+Channel",
        (object) "READ Mapping MAC+Channel",
        (object) "READ Device Info Delay",
        (object) "READ F/W Info",
        (object) "Write MAC+Channel",
        (object) "Write Mapping MAC+Channel",
        (object) "Write Device Info Delay",
        (object) "Copy to default area",
        (object) "Default area data comparison",
        (object) "READ MAC",
        (object) "READ Channel",
        (object) "Write MAC",
        (object) "Write Channel",
        (object) "Write Mapping MAC",
        (object) "Write Mapping Channel"
      });
      this.Type_Obj.Location = new Point(138, 62);
      this.Type_Obj.Name = "Type_Obj";
      this.Type_Obj.Size = new Size(252, 21);
      this.Type_Obj.TabIndex = 0;
      this.Type_Obj.SelectedIndexChanged += new EventHandler(this.Type_Obj_SelectedIndexChanged);
      this.label1.AutoSize = true;
      this.label1.Location = new Point(18, 26);
      this.label1.Name = "label1";
      this.label1.Size = new Size(34, 14);
      this.label1.TabIndex = 1;
      this.label1.Text = "Port";
      this.label2.AutoSize = true;
      this.label2.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.label2.Location = new Point(42, 234);
      this.label2.Name = "label2";
      this.label2.Size = new Size(38, 14);
      this.label2.TabIndex = 1;
      this.label2.Text = "MAC";
      this.label3.AutoSize = true;
      this.label3.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.label3.Location = new Point(41, 270);
      this.label3.Name = "label3";
      this.label3.Size = new Size(60, 14);
      this.label3.TabIndex = 1;
      this.label3.Text = "Channel";
      this.MAC_Obj.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.MAC_Obj.Location = new Point(140, 231);
      this.MAC_Obj.Name = "MAC_Obj";
      this.MAC_Obj.Size = new Size(153, 23);
      this.MAC_Obj.TabIndex = 2;
      this.MAC_Obj.TextAlign = HorizontalAlignment.Center;
      this.CH_Obj.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.CH_Obj.Location = new Point(140, 269);
      this.CH_Obj.Name = "CH_Obj";
      this.CH_Obj.Size = new Size(153, 23);
      this.CH_Obj.TabIndex = 2;
      this.CH_Obj.TextAlign = HorizontalAlignment.Center;
      this.GO_Obj.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.GO_Obj.Location = new Point(37, 455);
      this.GO_Obj.Name = "GO_Obj";
      this.GO_Obj.Size = new Size(169, 32);
      this.GO_Obj.TabIndex = 3;
      this.GO_Obj.Text = "SET";
      this.GO_Obj.UseVisualStyleBackColor = true;
      this.GO_Obj.Click += new EventHandler(this.GO_Obj_Click);
      this.History_Obj.Font = new Font("굴림", 8f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.History_Obj.Location = new Point(37, 495);
      this.History_Obj.Name = "History_Obj";
      this.History_Obj.Size = new Size(370, 150);
      this.History_Obj.TabIndex = 4;
      this.History_Obj.Text = "";
      this.MACResult_Obj.BackColor = SystemColors.ButtonFace;
      this.MACResult_Obj.BorderStyle = BorderStyle.FixedSingle;
      this.MACResult_Obj.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.MACResult_Obj.Location = new Point(315, 228);
      this.MACResult_Obj.Name = "MACResult_Obj";
      this.MACResult_Obj.Size = new Size(78, 29);
      this.MACResult_Obj.TabIndex = 5;
      this.MACResult_Obj.Text = "READY";
      this.MACResult_Obj.TextAlign = ContentAlignment.MiddleCenter;
      this.MACResult_Obj.Click += new EventHandler(this.MACResult_Obj_Click);
      this.CHResult_Obj.BackColor = SystemColors.ButtonFace;
      this.CHResult_Obj.BorderStyle = BorderStyle.FixedSingle;
      this.CHResult_Obj.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.CHResult_Obj.Location = new Point(315, 267);
      this.CHResult_Obj.Name = "CHResult_Obj";
      this.CHResult_Obj.Size = new Size(78, 29);
      this.CHResult_Obj.TabIndex = 5;
      this.CHResult_Obj.Text = "READY";
      this.CHResult_Obj.TextAlign = ContentAlignment.MiddleCenter;
      this.label5.AutoSize = true;
      this.label5.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.label5.Location = new Point(41, 309);
      this.label5.Name = "label5";
      this.label5.Size = new Size(80, 14);
      this.label5.TabIndex = 6;
      this.label5.Text = "Delay(sec)";
      this.Delay_Obj.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.Delay_Obj.Location = new Point(140, 308);
      this.Delay_Obj.Name = "Delay_Obj";
      this.Delay_Obj.Size = new Size(153, 23);
      this.Delay_Obj.TabIndex = 7;
      this.Delay_Obj.TextAlign = HorizontalAlignment.Center;
      this.button1.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.button1.Location = new Point(238, 454);
      this.button1.Name = "button1";
      this.button1.Size = new Size(169, 32);
      this.button1.TabIndex = 8;
      this.button1.Text = "GET";
      this.button1.UseVisualStyleBackColor = true;
      this.button1.Click += new EventHandler(this.button1_Click_1);
      this.MAP_MAC.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.MAP_MAC.Location = new Point(208, 346);
      this.MAP_MAC.Name = "MAP_MAC";
      this.MAP_MAC.Size = new Size(86, 23);
      this.MAP_MAC.TabIndex = 10;
      this.MAP_MAC.TextAlign = HorizontalAlignment.Center;
      this.label6.AutoSize = true;
      this.label6.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.label6.Location = new Point(43, 353);
      this.label6.Name = "label6";
      this.label6.Size = new Size(131, 14);
      this.label6.TabIndex = 9;
      this.label6.Text = "MAPPING RX MAC";
      this.label7.AutoSize = true;
      this.label7.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.label7.Location = new Point(43, 388);
      this.label7.Name = "label7";
      this.label7.Size = new Size(153, 14);
      this.label7.TabIndex = 11;
      this.label7.Text = "MAPPING RX Channel";
      this.MAP_CH.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.MAP_CH.Location = new Point(208, 385);
      this.MAP_CH.Name = "MAP_CH";
      this.MAP_CH.Size = new Size(86, 23);
      this.MAP_CH.TabIndex = 12;
      this.MAP_CH.TextAlign = HorizontalAlignment.Center;
      this.label8.AutoSize = true;
      this.label8.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.label8.Location = new Point(42, 197);
      this.label8.Name = "label8";
      this.label8.Size = new Size(65, 14);
      this.label8.TabIndex = 13;
      this.label8.Text = "F/W Ver.";
      this.FW_Ver.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.FW_Ver.Location = new Point(139, 193);
      this.FW_Ver.Name = "FW_Ver";
      this.FW_Ver.ReadOnly = true;
      this.FW_Ver.Size = new Size(153, 23);
      this.FW_Ver.TabIndex = 14;
      this.FW_Ver.TextAlign = HorizontalAlignment.Center;
      this.label9.AutoSize = true;
      this.label9.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.label9.Location = new Point(43, 426);
      this.label9.Name = "label9";
      this.label9.Size = new Size(199, 14);
      this.label9.TabIndex = 16;
      this.label9.Text = "Default area data comparison";
      this.DATA_COMPARE.Font = new Font("굴림", 10f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.DATA_COMPARE.Location = new Point(248, 422);
      this.DATA_COMPARE.Name = "DATA_COMPARE";
      this.DATA_COMPARE.ReadOnly = true;
      this.DATA_COMPARE.Size = new Size(46, 23);
      this.DATA_COMPARE.TabIndex = 17;
      this.DATA_COMPARE.TextAlign = HorizontalAlignment.Center;
      this.AutoScaleDimensions = new SizeF(10f, 19f);
      this.AutoScaleMode = AutoScaleMode.Font;
      this.BackColor = SystemColors.ButtonHighlight;
      this.ClientSize = new Size(449, 658);
      this.Controls.Add((Control) this.DATA_COMPARE);
      this.Controls.Add((Control) this.label9);
      this.Controls.Add((Control) this.FW_Ver);
      this.Controls.Add((Control) this.label8);
      this.Controls.Add((Control) this.MAP_CH);
      this.Controls.Add((Control) this.label7);
      this.Controls.Add((Control) this.MAP_MAC);
      this.Controls.Add((Control) this.label6);
      this.Controls.Add((Control) this.button1);
      this.Controls.Add((Control) this.Delay_Obj);
      this.Controls.Add((Control) this.label5);
      this.Controls.Add((Control) this.CHResult_Obj);
      this.Controls.Add((Control) this.MACResult_Obj);
      this.Controls.Add((Control) this.History_Obj);
      this.Controls.Add((Control) this.GO_Obj);
      this.Controls.Add((Control) this.CH_Obj);
      this.Controls.Add((Control) this.MAC_Obj);
      this.Controls.Add((Control) this.label3);
      this.Controls.Add((Control) this.label2);
      this.Controls.Add((Control) this.groupBox1);
      this.Font = new Font("굴림", 14.25f, FontStyle.Regular, GraphicsUnit.Point, (byte) 129);
      this.FormBorderStyle = FormBorderStyle.FixedDialog;
      this.KeyPreview = true;
      this.Margin = new Padding(4, 5, 4, 5);
      this.MaximizeBox = false;
      this.Name = nameof (Form1);
      this.Text = "Zigbee Setting_240326";
      this.Load += new EventHandler(this.Form1_Load);
      this.KeyDown += new KeyEventHandler(this.Form1_KeyDown);
      this.groupBox1.ResumeLayout(false);
      this.groupBox1.PerformLayout();
      this.ResumeLayout(false);
      this.PerformLayout();
    }

    private delegate void MyDelegate();
  }
}

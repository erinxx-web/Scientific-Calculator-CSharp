# Scientific-Calculator-CSharp

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Reflection;
using System.Runtime.InteropServices;
using System.Text;
using System.Text.RegularExpressions;
using System.Threading.Tasks;
using System.Windows.Forms;
using System.Media;
using System.IO;
using System.Speech.Synthesis;

namespace Calculator_242360R
{
    public partial class Form1 : Form
    {
        double currentValue = 0;
        double lastAnswer = 0;
        string currentOperator = "";
        bool isNewInput = true;
        bool isDegrees = true;
        bool audioOn = false;

        public Form1()
        {
            InitializeComponent();
        }

        protected override bool ProcessCmdKey(ref Message msg, Keys keyData)
        {
            switch (keyData)
            {
                case Keys.Enter: btnEqual.PerformClick(); return true;
                case Keys.C: btnAC.PerformClick(); return true;
                case Keys.Back: btnDEL.PerformClick(); return true;
                case Keys.D0: case Keys.NumPad0: btn0.PerformClick(); return true;
                case Keys.D1: case Keys.NumPad1: btn1.PerformClick(); return true;
                case Keys.D2: case Keys.NumPad2: btn2.PerformClick(); return true;
                case Keys.D3: case Keys.NumPad3: btn3.PerformClick(); return true;
                case Keys.D4: case Keys.NumPad4: btn4.PerformClick(); return true;
                case Keys.D5: case Keys.NumPad5: btn5.PerformClick(); return true;
                case Keys.D6: case Keys.NumPad6: btn6.PerformClick(); return true;
                case Keys.D7: case Keys.NumPad7: btn7.PerformClick(); return true;
                case Keys.D8: case Keys.NumPad8: btn8.PerformClick(); return true;
                case Keys.D9: case Keys.NumPad9: btn9.PerformClick(); return true;
                case Keys.Add: case Keys.Oemplus: btnAdd.PerformClick(); return true;
                case Keys.Subtract: case Keys.OemMinus: btnsubtract.PerformClick(); return true;
                case Keys.Multiply: btnMultiply.PerformClick(); return true;
                case Keys.Divide: case Keys.OemQuestion: btnDivide.PerformClick(); return true;
                case Keys.Oem5: btnMod.PerformClick(); return true;
            }
            return base.ProcessCmdKey(ref msg, keyData);
        }

        private void PlayClickSound()
        {
            if (!audioOn) return;
            try
            {
                using (var stream = new MemoryStream(Properties.Resources.ClickSound))
                {
                    SoundPlayer player = new SoundPlayer(stream);
                    player.Play();
                }
            }
            catch { }
        }

        private void SpeakResult(string text)
        {
            if (!audioOn) return;

            try
            {
                SpeechSynthesizer synth = new SpeechSynthesizer();
                synth.SpeakAsync(text);
            }
            catch
            { }
        }

        private void AppendDigit(string digit)
        {
            if (digit == "." && lblResult.Text.Contains("."))
                return; 
            PlayClickSound();
            if (lblResult.Text == "0" || isNewInput)
                lblResult.Text = digit;
            else
                lblResult.Text += digit;

            lblEqn.Text += digit;
            isNewInput = false;
            if (digit == "." && lblResult.Text.Contains(".")) return;

            EvaluateFullExpression();

        }

        private void SetOperator(string op)
        {
            PlayClickSound();
            if (string.IsNullOrWhiteSpace(lblEqn.Text)) return;

            if (Regex.IsMatch(lblEqn.Text.Trim(), @"[+\-*/%]\s?$"))
            {
                lblEqn.Text = Regex.Replace(lblEqn.Text.Trim(), @"[+\-*/%]\s?$", op);
            }
            else
            {
                lblEqn.Text += " " + op + " ";
                currentValue = Convert.ToDouble(lblResult.Text);
                currentOperator = op;
                isNewInput = true;

                EvaluateFullExpression();
            }
        }

        private void btnEqual_Click(object sender, EventArgs e)
        {
            PlayClickSound();

            string expr = lblEqn.Text.Replace(" ", "");

            expr = expr.Replace("×", "*").Replace("÷", "/");

            while (expr.Length > 0 && "+-*/%".Contains(expr.Last()))
            {
                expr = expr.Substring(0, expr.Length - 1);
            }

            if (string.IsNullOrEmpty(expr))
            {
                lblResult.Text = "0";
                return;
            }

            try
            {
                DataTable dt = new DataTable();
                var val = dt.Compute(expr, "");
                double result = Convert.ToDouble(val);

                lblResult.Text = result.ToString("0.#####");
                SpeakResult(lblResult.Text);

                lastAnswer = result;
                currentValue = result;
                currentOperator = "";
                lblEqn.Text = "";
                isNewInput = true;
            }
            catch
            {
            }
        }

        private void btnAC_Click(object sender, EventArgs e)
        {
            lblResult.Text = "0";
            lblEqn.Text = "";
            currentValue = 0;
            currentOperator = "";
            isNewInput = true;
        }

        private void btnDEL_Click(object sender, EventArgs e)
        {
            if (!isNewInput && lblResult.Text.Length > 1)
                lblResult.Text = lblResult.Text.Substring(0, lblResult.Text.Length - 1);
            else
                lblResult.Text = "0";

            if (lblEqn.Text.Length > 1)
                lblEqn.Text = lblEqn.Text.Substring(0, lblResult.Text.Length - 1);
            else
                lblEqn.Text = "";

            if (string.IsNullOrWhiteSpace(lblEqn.Text))
                lblResult.Text = "0";
        }

        private void btnANS_Click(object sender, EventArgs e)
        {
            lblResult.Text = lastAnswer.ToString("0.#####");
            isNewInput = true;
        }

        private void btnCOPY_Click(object sender, EventArgs e)
        {
            Clipboard.SetText(lblResult.Text);
        }

        private void btnSPK_Click(object sender, EventArgs e)
        {
            audioOn = !audioOn;
            btnSPK.Text = audioOn ? "SPK ON" : "SPK OFF";
        }

        private void btnDEG_Click(object sender, EventArgs e)
        {
            isDegrees = !isDegrees;
            btnDEG.Text = isDegrees ? "DEG" : "RAD";
        }

        private void btnSHIFT_Click(object sender, EventArgs e)
        {
            btnSHIFT.BackColor = btnSHIFT.BackColor == Color.Orange ? SystemColors.Control : Color.Orange;
        }

        private double ToRadians(double angle) => isDegrees ? angle * (Math.PI / 180) : angle;

        private void ApplyUnary(Func<double, double> func)
        {
            if (double.TryParse(lblResult.Text, out double value))
            {
                double result = func(value);
                lblResult.Text = result.ToString("0.#####");
                lastAnswer = result;
                isNewInput = true;
            }
        }

        private void ApplyUnary(Func<double, double, double> func, double operand)
        {
            if (double.TryParse(lblResult.Text, out double value))
            {
                double result = func(value, operand);
                lblResult.Text = result.ToString("0.#####");
                lastAnswer = result;
                isNewInput = true;
                SpeakResult(lblResult.Text);
            }
        }

        private void EvaluateFullExpression()
        {
            string expr = lblEqn.Text.Replace(" ", ""); 

            while (expr.Length > 0 && "+-*/%".Contains(expr.Last()))
            {
                expr = expr.Substring(0, expr.Length - 1);
            }

            if (string.IsNullOrEmpty(expr))
            {
                lblResult.Text = "0";
                return;
            }

            try
            {
                DataTable dt = new DataTable();
                var val = dt.Compute(expr, "");
                lblResult.Text = Convert.ToDouble(val).ToString("0.#####");
            }
            catch
            {
            }
        }

        private void btn0_Click(object sender, EventArgs e) => AppendDigit("0");
        private void btn1_Click(object sender, EventArgs e) => AppendDigit("1");
        private void btn2_Click(object sender, EventArgs e) => AppendDigit("2");
        private void btn3_Click(object sender, EventArgs e) => AppendDigit("3");
        private void btn4_Click(object sender, EventArgs e) => AppendDigit("4");
        private void btn5_Click(object sender, EventArgs e) => AppendDigit("5");
        private void btn6_Click(object sender, EventArgs e) => AppendDigit("6");
        private void btn7_Click(object sender, EventArgs e) => AppendDigit("7");
        private void btn8_Click(object sender, EventArgs e) => AppendDigit("8");
        private void btn9_Click(object sender, EventArgs e) => AppendDigit("9");
        private void btnDot_Click(object sender, EventArgs e) => AppendDigit(".");

        private void btnAdd_Click(object sender, EventArgs e) => SetOperator("+");
        private void btnSubtract_Click(object sender, EventArgs e) => SetOperator("-");
        private void btnMultiply_Click(object sender, EventArgs e) => SetOperator("*");
        private void btnDivide_Click(object sender, EventArgs e) => SetOperator("/");
        private void btnMod_Click(object sender, EventArgs e) => SetOperator("%");

        private void btnNegate_Click(object sender, EventArgs e) => ApplyUnary(x => -x);
        private void btnSquare_Click(object sender, EventArgs e) => ApplyUnary(x => Math.Pow(x, 2));
        private void btnSqrt_Click(object sender, EventArgs e) => ApplyUnary(Math.Sqrt);
        private void btnReciprocal_Click(object sender, EventArgs e) => ApplyUnary(x => 1 / x);
        private void btnLog10_Click(object sender, EventArgs e) => ApplyUnary(Math.Log10);
        private void btnLn_Click(object sender, EventArgs e) => ApplyUnary(Math.Log);
        private void btnPow10_Click(object sender, EventArgs e) => ApplyUnary(x => Math.Pow(10, x));
        private void btnExp_Click(object sender, EventArgs e) => ApplyUnary(Math.Exp);
        private void btnSin_Click(object sender, EventArgs e) => ApplyUnary(x => Math.Sin(ToRadians(x)));
        private void btnCos_Click(object sender, EventArgs e) => ApplyUnary(x => Math.Cos(ToRadians(x)));
        private void btnTan_Click(object sender, EventArgs e) => ApplyUnary(x => Math.Tan(ToRadians(x)));
    }
}

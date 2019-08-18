using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Windows.Forms;
using System.Data.OleDb;
using MySql.Data.MySqlClient;
//gak usah perhatiin yang ini. cuma manggil library aja

namespace SOM_Eucledian
{
   public partial class Main : Form
    {
       
        
        string connectionSQL = "server=localhost;database=data;uid=root;password=;";

        private void FormClosingEventCancle_Closing(object sender, System.ComponentModel.CancelEventArgs e)
        {
            DialogResult dr = MessageBox.Show("Terimaksih telah menggunakan aplikasi ini. \n Pastikan kelengkapan data telah Anda peroleh. \n \n Semoga bermanfaat!",
              "Prasetyo, Siswo Budi said :", MessageBoxButtons.YesNo);
            if (dr == DialogResult.No)
                e.Cancel = true;
            else
                e.Cancel = false;
        }
       
       public Main()
        {
            InitializeComponent();
            this.Closing += new System.ComponentModel.CancelEventHandler(this.FormClosingEventCancle_Closing);

        }
     
        private void Main_Load(object sender, EventArgs e)
        {
            int i = 0;
            MySqlConnection db = new MySqlConnection(connectionSQL);
            db.Open();
            MySqlCommand dbcmd = db.CreateCommand();
            string sql = "show tables";
            dbcmd.CommandText = sql;
            MySqlDataReader reader = dbcmd.ExecuteReader();
            while (reader.Read())
            {
                comboBox1.Items.Add(reader.GetString(0));
                comboBox3.Items.Add(reader.GetString(0));
                i = i + 1;
            }
            
            MySqlConnection db2 = new MySqlConnection(connectionSQL);
            db2.Open();

            MySqlCommand dbcmd2 = db2.CreateCommand();
            string sql2 = "select propinsi from propinsi";
            dbcmd2.CommandText = sql2;
            MySqlDataReader reader2 = dbcmd2.ExecuteReader();
            while (reader2.Read())
            {
                comboBox4.Items.Add(reader2.GetString(0));
                i = i + 1;
            }
            
            db2.Close();
            
        }

       
          
       public void comboBox1_SelectedIndexChanged_1(object sender, EventArgs e)
        {
            MySqlConnection db = new MySqlConnection(connectionSQL);
            db.Open();
            MySqlCommand dbcmd = db.CreateCommand();
            string sql = "select * from " + comboBox1.SelectedItem.ToString();
            MySqlDataAdapter DA = new MySqlDataAdapter(sql, connectionSQL);
            DataTable table = new DataTable();
            DA.Fill(table);
            dataGridView1.DataSource = table;
            db.Close();
        }

       public void binding()
       {
           try
           {
               MySqlConnection data = new MySqlConnection(connectionSQL);
               data.Open();
               //MySqlCommand dbcmd = data.CreateCommand();
               string sql = "select * from " + comboBox3.SelectedItem.ToString();
               MySqlDataAdapter DAku = new MySqlDataAdapter(sql, connectionSQL);
               MySqlCommandBuilder comman = new MySqlCommandBuilder(DAku);
               DataTable neotable = new DataTable();
               DAku.Fill(neotable);
               BindingSource bindme = new BindingSource();
               bindme.DataSource = neotable;
               dataGridView3.DataSource = bindme;
               data.Close();
           }
           catch (Exception err) { MessageBox.Show(err.Message); }
          
       }


        private void comboBox3_SelectedIndexChanged(object sender, EventArgs e)
        {
            binding();
           
        }

        private void button3_Click_1(object sender, EventArgs e)
        {
            dataGridView2.Rows.Clear();
            dataGridView4.Rows.Clear();
            dataGridView5.Rows.Clear();
            dataGridView9.Rows.Clear();
            dataGridView10.Rows.Clear();

           if (radioButton1.Checked)
           {
               SOM();
           }
           else if (radioButton2.Checked)
           {
               SOM_Eucledian();
           }
           else if (radioButton3.Checked)
           {
               SOM_Manhattan();
           }
           else
           {
               MessageBox.Show("Mungkin Anda belum memilih Methode apa yang akan digunakan dan atau melupakan beberapa parameter yang harus diisi terlebih dahulu.  \n  \nUntuk itu, mohon ulangi sekali lagi");
           }
        }
           
       
        public void SOM()
        {
                      
            int rowCount = ((DataTable)this.dataGridView1.DataSource).Rows.Count;
            int columnCount = ((DataTable)this.dataGridView1.DataSource).Columns.Count;
            double[,] c = new double[rowCount, columnCount];
            double[] terbesar = new double[columnCount];
            double[] terkecil = new double[columnCount];
            double[] hasil = new double[rowCount];
            double[] bobot = new double[columnCount];
            double[] bobot_temp = new double[columnCount];
            int[] cluster = new int[rowCount];
            double[] win = new double[11];
            string[] nama_cluster = new string[rowCount];
            double MSE = 2;
            double lerning, bil_mse, min, max, selisih, b1, b2, b3, b4, b5, b6, b7, b8, b9, erorr = 0;
            int iterasi = 1;
            int acak1, acak2, baris_sembarang, jumlah_cluster = 0;
            Random acak = new Random();
            Random acak_baris = new Random();
            //pemindahan data dari datagrid ke array agar mudah diakses dan pancarian nilai max dan min per kolom
            for (int a = 0; a < rowCount; a++)
            {
                for (int b = 1; b < columnCount; b++)
                {
                    c[a, b] = Convert.ToDouble(dataGridView1.Rows[a].Cells[b].Value);
                }

            }

            dataGridView2.Rows.Add(rowCount);
            
            //pencarian nilai maksimum dan minimum per kolom
            for (int a = 1; a < columnCount; a++)
            {
                terbesar[a] = Math.Max(c[0, a], c[1, a]);
                terkecil[a] = Math.Min(c[0, a], c[1, a]);

                for (int b = 1; b < rowCount; b++)
                {
                    terbesar[a] = Math.Max(terbesar[a], c[b, a]);
                    terkecil[a] = Math.Min(terkecil[a], c[b, a]);
                }

             }
            //normalisasi data
            for (int a = 1; a < columnCount; a++)
            {
                for (int b = 0; b < rowCount; b++)
                {
                    c[b, a] = (c[b, a] - terkecil[a]) / (terbesar[a] - terkecil[a]);
                                       
                }
            }
            acak1 = Convert.ToInt16(comboBox5.SelectedItem.ToString());
            acak2 = Convert.ToInt16(comboBox6.SelectedItem.ToString());
            //pembuatan nilai bobot secara random
            for (int a = 1; a < columnCount; a++)
            {
                bobot[a] = acak.Next(acak1, acak2);
            }
			//pengambilan variable lerning dan MSE dari combo_box
            lerning = Convert.ToDouble(comboBox2.SelectedItem.ToString());
            erorr = Convert.ToDouble(comboBox7.SelectedItem.ToString());
            erorr = erorr / 100; //karena MSE itu persentase erorr makanya dibagi seratus
           
		   //nah, disini SOM dimulai
            while (MSE > erorr)
            {
                baris_sembarang = acak_baris.Next(0, rowCount - 1);
                for (int col = 1; col < columnCount; col++)
                {
                    bobot_temp[col] = bobot[col];
                  
                }
                for (int col = 1; col < columnCount; col++)
                {
                    bobot[col] = bobot[col] + lerning * (c[baris_sembarang, col] - bobot[col]);

                }
                bil_mse = 0;
                for (int col = 1; col < columnCount; col++)
                {
                    bil_mse = bil_mse + (Math.Pow(bobot_temp[col] - bobot[col], 2));
                }
                MSE = Math.Pow(bil_mse, 0.5);
                dataGridView4.Rows.Add(1);
                dataGridView4.Rows[iterasi - 1].Cells[0].Value = iterasi.ToString();
                dataGridView4.Rows[iterasi - 1].Cells[1].Value = lerning.ToString();
                dataGridView5.Rows.Add(1);
                dataGridView5.Rows[iterasi - 1].Cells[0].Value = iterasi.ToString();
                dataGridView5.Rows[iterasi - 1].Cells[1].Value = MSE.ToString();
                iterasi = iterasi + 1;
                lerning = lerning / 2;
                
            }
            for (int a = 0; a < rowCount; a++)
            {
                for (int b = 1; b < columnCount; b++)
                {
                    c[a, b] = c[a, b] * bobot[b];
                }
            }
            for (int a = 0; a < rowCount; a++)
            {
                hasil[a] = 0;
                for (int b = 1; b < columnCount; b++)
                {
                    hasil[a] = hasil[a] + c[a, b];
                }
                if (hasil[a] < 0)
                {
                    hasil[a] = hasil[a] * -1;
                }
                dataGridView9.Rows.Add(1);
                dataGridView9.Rows[a].Cells[0].Value = dataGridView1.Rows[a].Cells[0].Value;
                dataGridView9.Rows[a].Cells[1].Value = hasil[a].ToString();
            }
            max = Math.Max(hasil[0], hasil[1]);
            min = Math.Min(hasil[0], hasil[1]);
            for (int a = 2; a < rowCount; a++)
            {
                max = Math.Max(max, hasil[a]);
                min = Math.Min(min, hasil[a]);
            }

            selisih = max-min;

            jumlah_cluster = Convert.ToInt16(comboBox9.SelectedItem.ToString());
            //MessageBox.Show("max " + max.ToString() + "\n min " + min.ToString());

            if (jumlah_cluster == 3) // mulai mapping gan kalo clusternya 3
            {
                selisih = selisih / 3;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                win[1] = min;
                win[2] = (b2 - b1) / 2 + b1;
                win[3] = max;
               
                    for (int a = 0; a < rowCount; a++)
                    {
                        if (hasil[a] < b1)
                        {
                            cluster[a] = 1;
                            nama_cluster[a] = "Cluster Pertama";
                        }
                        else if (hasil[a] < b2 && hasil[a] > b1)
                        {
                            cluster[a] = 2;
                            nama_cluster[a] = "Cluster Kedua";
                        }
                        else if (hasil[a] <= max && hasil[a] > b2)
                        {
                            cluster[a] = 3;
                            nama_cluster[a] = "Cluster Ketiga";
                        }
                        else if (hasil[a] == b1 && hasil[a] == b2)
                        {
                            cluster[a] = 0;
                            nama_cluster[a] = "Outlier";
                        }
                    }
            }
            if (jumlah_cluster == 4) //kalau clusternya 4
            {
                selisih = selisih / 4;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                win[1] = min;
                win[2] = (b2 - b1) / 2 + b1;
                win[3] = (b3 - b2) / 2 + b2;
                win[4] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] < b1)
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] < b2 && hasil[a] > b1)
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] < b3 && hasil[a] > b2)
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] <= max && hasil[a] > b3)
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else if (hasil[a] == b1 && hasil[a] == b2 && hasil[a] == b3)
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            if (jumlah_cluster == 5)
            {
                selisih = selisih / 5;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                b4 = (4 * selisih) + min;
                win[1] = min;
                win[2] = (b2 - b1) / 2 + b1;
                win[3] = (b3 - b2) / 2 + b2;
                win[4] = (b4 - b3) / 2 + b3;
                win[5] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] < b1)
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] < b2 && hasil[a] > b1)
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] < b3 && hasil[a] > b2)
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] < b4 && hasil[a] > b3)
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else if (hasil[a] <= max && hasil[a] > b4)
                    {
                        cluster[a] = 5;
                        nama_cluster[a] = "Cluster Kelima";
                    }
                    else if (hasil[a] == b1 && hasil[a] == b2 && hasil[a] == b3 && hasil[a] == b4)
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            if (jumlah_cluster == 6)
            {
                selisih = selisih / 6;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                b4 = (4 * selisih) + min;
                b5 = (5 * selisih) + min;
                win[1] = min;
                win[2] = (b2 - b1) / 2 + b1;
                win[3] = (b3 - b2) / 2 + b2;
                win[4] = (b4 - b3) / 2 + b3;
                win[5] = (b5 - b4) / 2 + b4;
                win[6] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] < b1)
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] < b2 && hasil[a] > b1)
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] < b3 && hasil[a] > b2)
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] < b4 && hasil[a] > b3)
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else if (hasil[a] < b5 && hasil[a] > b4)
                    {
                        cluster[a] = 5;
                        nama_cluster[a] = "Cluster Kelima";
                    }
                    else if (hasil[a] <= max && hasil[a] > b5)
                    {
                        cluster[a] = 6;
                        nama_cluster[a] = "Cluster Keenam";
                    }
                    else if (hasil[a] == b1 && hasil[a] == b2 && hasil[a] == b3 && hasil[a] == b4 && hasil[a] == b5)
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            if (jumlah_cluster == 7)
            {
                selisih = selisih / 7;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                b4 = (4 * selisih) + min;
                b5 = (5 * selisih) + min;
                b6 = (6 * selisih) + min;
                win[1] = min;
                win[2] = (b2 - b1) / 2 + b1;
                win[3] = (b3 - b2) / 2 + b2;
                win[4] = (b4 - b3) / 2 + b3;
                win[5] = (b5 - b4) / 2 + b4;
                win[6] = (b6 - b5) / 2 + b5;
                win[7] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] < b1)
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] < b2 && hasil[a] > b1)
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] < b3 && hasil[a] > b2)
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] < b4 && hasil[a] > b3)
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else if (hasil[a] < b5 && hasil[a] > b4)
                    {
                        cluster[a] = 5;
                        nama_cluster[a] = "Cluster Kelima";
                    }
                    else if (hasil[a] < b6 && hasil[a] > b5)
                    {
                        cluster[a] = 6;
                        nama_cluster[a] = "Cluster Keenam";
                    }
                    else if (hasil[a] <= max && hasil[a] > b6)
                    {
                        cluster[a] = 7;
                        nama_cluster[a] = "Cluster Ketujuh";
                    }
                    else if (hasil[a] == b1 && hasil[a] == b2 && hasil[a] == b3 && hasil[a] == b4 && hasil[a] == b5 && hasil[a] == b6)
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            if (jumlah_cluster == 8)
            {
                selisih = selisih / 8;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                b4 = (4 * selisih) + min;
                b5 = (5 * selisih) + min;
                b6 = (6 * selisih) + min;
                b7 = (7 * selisih) + min;
                win[1] = min;
                win[2] = (b2 - b1) / 2 + b1;
                win[3] = (b3 - b2) / 2 + b2;
                win[4] = (b4 - b3) / 2 + b3;
                win[5] = (b5 - b4) / 2 + b4;
                win[6] = (b6 - b5) / 2 + b5;
                win[7] = (b7 - b6) / 2 + b6;
                win[8] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] < b1)
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] < b2 && hasil[a] > b1)
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] < b3 && hasil[a] > b2)
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] < b4 && hasil[a] > b3)
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else if (hasil[a] < b5 && hasil[a] > b4)
                    {
                        cluster[a] = 5;
                        nama_cluster[a] = "Cluster Kelima";
                    }
                    else if (hasil[a] < b6 && hasil[a] > b5)
                    {
                        cluster[a] = 6;
                        nama_cluster[a] = "Cluster Keenam";
                    }
                    else if (hasil[a] < b7 && hasil[a] > b6)
                    {
                        cluster[a] = 7;
                        nama_cluster[a] = "Cluster Ketujuh";
                    }
                    else if (hasil[a] <= max && hasil[a] > b7)
                    {
                        cluster[a] = 8;
                        nama_cluster[a] = "Cluster Kedelapan";
                    }
                    else if (hasil[a] == b1 && hasil[a] == b2 && hasil[a] == b3 && hasil[a] == b4 && hasil[a] == b5 && hasil[a] == b6 && hasil[a] == b7)
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            if (jumlah_cluster == 9)
            {
                selisih = selisih / 9;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                b4 = (4 * selisih) + min;
                b5 = (5 * selisih) + min;
                b6 = (6 * selisih) + min;
                b7 = (7 * selisih) + min;
                b8 = (8 * selisih) + min;
                win[1] = min;
                win[2] = (b2 - b1) / 2 + b1;
                win[3] = (b3 - b2) / 2 + b2;
                win[4] = (b4 - b3) / 2 + b3;
                win[5] = (b5 - b4) / 2 + b4;
                win[6] = (b6 - b5) / 2 + b5;
                win[7] = (b7 - b6) / 2 + b6;
                win[8] = (b8 - b7) / 2 + b7;
                win[9] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] < b1)
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] < b2 && hasil[a] > b1)
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] < b3 && hasil[a] > b2)
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] < b4 && hasil[a] > b3)
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else if (hasil[a] < b5 && hasil[a] > b4)
                    {
                        cluster[a] = 5;
                        nama_cluster[a] = "Cluster Kelima";
                    }
                    else if (hasil[a] < b6 && hasil[a] > b5)
                    {
                        cluster[a] = 6;
                        nama_cluster[a] = "Cluster Keenam";
                    }
                    else if (hasil[a] < b7 && hasil[a] > b6)
                    {
                        cluster[a] = 7;
                        nama_cluster[a] = "Cluster Ketujuh";
                    }
                    else if (hasil[a] < b8 && hasil[a] > b7)
                    {
                        cluster[a] = 8;
                        nama_cluster[a] = "Cluster Kedelapan";
                    }
                    else if (hasil[a] <= max && hasil[a] > b8)
                    {
                        cluster[a] = 9;
                        nama_cluster[a] = "Cluster Kesembilan";
                    }
                    else if (hasil[a] == b1 && hasil[a] == b2 && hasil[a] == b3 && hasil[a] == b4 && hasil[a] == b5 && hasil[a] == b6 && hasil[a] == b7 && hasil[a] == b8)
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            if (jumlah_cluster == 10)
            {
                selisih = selisih / 10;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                b4 = (4 * selisih) + min;
                b5 = (5 * selisih) + min;
                b6 = (6 * selisih) + min;
                b7 = (7 * selisih) + min;
                b8 = (8 * selisih) + min;
                b9 = (9 * selisih) + min;
                win[1] = min;
                win[2] = (b2 - b1) / 2 + b1;
                win[3] = (b3 - b2) / 2 + b2;
                win[4] = (b4 - b3) / 2 + b3;
                win[5] = (b5 - b4) / 2 + b4;
                win[6] = (b6 - b5) / 2 + b5;
                win[7] = (b7 - b6) / 2 + b6;
                win[8] = (b8 - b7) / 2 + b7;
                win[9] = (b9 - b8) / 2 + b8;
                win[10] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] < b1)
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] < b2 && hasil[a] > b1)
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] < b3 && hasil[a] > b2)
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] < b4 && hasil[a] > b3)
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else if (hasil[a] < b5 && hasil[a] > b4)
                    {
                        cluster[a] = 5;
                        nama_cluster[a] = "Cluster Kelima";
                    }
                    else if (hasil[a] < b6 && hasil[a] > b5)
                    {
                        cluster[a] = 6;
                        nama_cluster[a] = "Cluster Keenam";
                    }
                    else if (hasil[a] < b7 && hasil[a] > b6)
                    {
                        cluster[a] = 7;
                        nama_cluster[a] = "Cluster Ketujuh";
                    }
                    else if (hasil[a] < b8 && hasil[a] > b7)
                    {
                        cluster[a] = 8;
                        nama_cluster[a] = "Cluster Kedelapan";
                    }
                    else if (hasil[a] < b9 && hasil[a] > b8)
                    {
                        cluster[a] = 9;
                        nama_cluster[a] = "Cluster Kesembilan";
                    }
                    else if (hasil[a] <= max && hasil[a] > b9)
                    {
                        cluster[a] = 10;
                        nama_cluster[a] = "Cluster Kesepuluh";
                    }
                    else if (hasil[a] == b1 && hasil[a] == b2 && hasil[a] == b3 && hasil[a] == b4 && hasil[a] == b5 && hasil[a] == b6 && hasil[a] == b7 && hasil[a] == b8 && hasil[a] == b9)
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            for (int a = 0; a < jumlah_cluster; a++)
            {
                dataGridView10.Rows.Add(1);
                dataGridView10.Rows[a].Cells[0].Value = (a + 1).ToString();
                dataGridView10.Rows[a].Cells[1].Value = win[a + 1].ToString();
            }
            
            for (int a = 0; a < rowCount; a++)
            {
                dataGridView2.Rows[a].Cells[0].Value = dataGridView1.Rows[a].Cells[0].Value;
                dataGridView2.Rows[a].Cells[1].Value = cluster[a].ToString();
                dataGridView2.Rows[a].Cells[2].Value = nama_cluster[a].ToString();
            }
            MessageBox.Show("Naive SOM");
        }

        public void SOM_Eucledian()
        {
            int rowCount = ((DataTable)this.dataGridView1.DataSource).Rows.Count;
            int columnCount = ((DataTable)this.dataGridView1.DataSource).Columns.Count;
            double[,] c = new double[rowCount, columnCount];
            double[,] d = new double[rowCount, columnCount];
            double[] terbesar = new double[columnCount];
            double[] terkecil = new double[columnCount];
            double[] hasil = new double[rowCount];
            double[] bobot = new double[columnCount];
            double[] bobot_awal = new double[columnCount];
            double[] bobot_temp = new double[columnCount];
            double[] win = new double[11];
            int[] cluster = new int[rowCount];
            string[] nama_cluster = new string[rowCount];
            double MSE = 2;
            double lerning, bil_mse, min, max, selisih, b1, b2, b3, b4, b5, b6, b7, b8, b9, erorr, distance = 0;
            int iterasi = 1;
            int acak1, acak2, baris_sembarang, jumlah_cluster = 0;
            Random acak = new Random();
            Random acak_baris = new Random();
            //pemindahan data dari datagrid ke array agar mudah diakses dan pancarian nilai max dan min per kolom
            for (int a = 0; a < rowCount; a++)
            {
                for (int b = 1; b < columnCount; b++)
                {
                    c[a, b] = Convert.ToDouble(dataGridView1.Rows[a].Cells[b].Value);
                    d[a, b] = Convert.ToDouble(dataGridView1.Rows[a].Cells[b].Value);
                }

            }

            dataGridView2.Rows.Add(rowCount);
            //pencarian nilai maksimum dan minimum per kolom
            for (int a = 1; a < columnCount; a++)
            {
                terbesar[a] = Math.Max(c[0, a], c[1, a]);
                terkecil[a] = Math.Min(c[0, a], c[1, a]);

                for (int b = 1; b < rowCount; b++)
                {
                    terbesar[a] = Math.Max(terbesar[a], c[b, a]);
                    terkecil[a] = Math.Min(terkecil[a], c[b, a]);
                }

            }
            //normalisasi data
            for (int a = 1; a < columnCount; a++)
            {
                for (int b = 0; b < rowCount; b++)
                {
                    c[b, a] = (c[b, a] - terkecil[a]) / (terbesar[a] - terkecil[a]);
                                             
                }
            }
            acak1 = Convert.ToInt16(comboBox5.SelectedItem.ToString());
            acak2 = Convert.ToInt16(comboBox6.SelectedItem.ToString());
            //pembuatan nilai bobot secara random
            for (int a = 1; a < columnCount; a++)
            {
                bobot[a] = acak.Next(acak1, acak2);
            }

            for (int a = 1; a < columnCount; a++)
            {
                bobot_awal[a] = bobot[a];
            }

            lerning = Convert.ToDouble(comboBox2.SelectedItem.ToString());
            erorr = Convert.ToDouble(comboBox7.SelectedItem.ToString());
            erorr = erorr / 100;
            while (MSE > erorr)
            {
                baris_sembarang = acak_baris.Next(0, rowCount - 1);
                for (int col = 1; col < columnCount; col++)
                {
                    bobot_temp[col] = bobot[col];
                    
                }
                for (int col = 1; col < columnCount; col++)
                {
                    bobot_awal[col] = bobot[col];
                    bobot[col] = bobot[col] + lerning * (c[baris_sembarang, col] - bobot[col]);

                }
                bil_mse = 0;
                for (int col = 1; col < columnCount; col++)
                {
                    bil_mse = bil_mse + (Math.Pow(bobot_temp[col] - bobot[col], 2));
                }
                MSE = Math.Pow(bil_mse, 0.5);
                dataGridView4.Rows.Add(1);
                dataGridView4.Rows[iterasi - 1].Cells[0].Value = iterasi.ToString();
                dataGridView4.Rows[iterasi - 1].Cells[1].Value = lerning.ToString();
                dataGridView5.Rows.Add(1);
                dataGridView5.Rows[iterasi - 1].Cells[0].Value = iterasi.ToString();
                dataGridView5.Rows[iterasi - 1].Cells[1].Value = MSE.ToString();
                iterasi = iterasi + 1;
                lerning = lerning / 2;
            }
            for (int a = 0; a < rowCount; a++)
            {
                for (int b = 1; b < columnCount; b++)
                {
                    c[a, b] = c[a, b] * bobot[b];
                }
            }
            for (int a = 0; a < rowCount; a++)
            {
                for (int b = 1; b < columnCount; b++)
                {
                    d[a, b] = d[a, b] * bobot_awal[b];
                }
            }
            for (int a = 0; a < rowCount; a++)
            {
                hasil[a] = 0;
                for (int b = 1; b < columnCount; b++)
                {
                    hasil[a] = hasil[a] + Math.Pow((c[a, b] - d[a, b]), 2);
                }
                hasil[a] = Math.Pow(hasil[a], 0.5);
                /*if (hasil[a] < 0)
                {
                    hasil[a] = hasil[a] * -1;
                }*/
                dataGridView9.Rows.Add(1);
                dataGridView9.Rows[a].Cells[0].Value = dataGridView1.Rows[a].Cells[0].Value;
                dataGridView9.Rows[a].Cells[1].Value = hasil[a].ToString();
            }
            max = Math.Max(hasil[0], hasil[1]);
            min = Math.Min(hasil[0], hasil[1]);
            for (int a = 2; a < rowCount; a++)
            {
                max = Math.Max(max, hasil[a]);
                min = Math.Min(min, hasil[a]);
            }

            distance = Convert.ToInt16(comboBox8.SelectedItem.ToString());
            selisih = max - min;
            jumlah_cluster = Convert.ToInt16(comboBox9.SelectedItem.ToString());
            //MessageBox.Show("max " + max.ToString() + "\n min " + min.ToString() + "\n dist " + distance.ToString() + "\n selisih " + selisih.ToString());

            if (jumlah_cluster == 3)
            {
                selisih = selisih / 3;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                win[1] = min;
                win[2] = ((b2 - b1) / 2) + b1;
                win[3] = max;
               
                
                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] <= ( win[1] + distance ))
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] >= (win[2] - distance) && hasil[a] <= (win[2] + distance))
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] >= ( win[3] - distance ) )
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }
            if (jumlah_cluster == 4)
            {
                selisih = selisih / 4;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                win[1] = min;
                win[2] = ((b2 - b1) / 2) + b1;
                win[3] = ((b3 - b2) / 2) + b2;
                win[4] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] <= (win[1] + distance))
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] >= (win[2] - distance) && hasil[a] <= (win[2] + distance))
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] >= (win[3] - distance) && hasil[a] <= (win[3] + distance))
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] >= (win[4] - distance))
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else 
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            if (jumlah_cluster == 5)
            {
                selisih = selisih / 5;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                b4 = (4 * selisih) + min;
                win[1] = min;
                win[2] = ((b2 - b1) / 2) + b1;
                win[3] = ((b3 - b2) / 2) + b2;
                win[4] = ((b4 - b3) / 2) + b3;
                win[5] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] <= (win[1] + distance))
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] >= (win[2] - distance) && hasil[a] <= (win[2] + distance))
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] >= (win[3] - distance) && hasil[a] <= (win[3] + distance))
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] >= (win[4] - distance) && hasil[a] <= (win[4] + distance))
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else if (hasil[a] >= (win[5] - distance))
                    {
                        cluster[a] = 5;
                        nama_cluster[a] = "Cluster Kelima";
                    }
                    else
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            if (jumlah_cluster == 6)
            {
                selisih = selisih / 6;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                b4 = (4 * selisih) + min;
                b5 = (5 * selisih) + min;
                win[1] = min;
                win[2] = ((b2 - b1) / 2) + b1;
                win[3] = ((b3 - b2) / 2) + b2;
                win[4] = ((b4 - b3) / 2) + b3;
                win[5] = ((b5 - b4) / 2) + b4;
                win[6] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] <= (win[1] + distance))
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] >= (win[2] - distance) && hasil[a] <= (win[2] + distance))
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] >= (win[3] - distance) && hasil[a] <= (win[3] + distance))
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] >= (win[4] - distance) && hasil[a] <= (win[4] + distance))
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else if (hasil[a] >= (win[5] - distance) && hasil[a] <= (win[5] + distance))
                    {
                        cluster[a] = 5;
                        nama_cluster[a] = "Cluster Kelima";
                    }
                    else if (hasil[a] >= (win[6] - distance))
                    {
                        cluster[a] = 6;
                        nama_cluster[a] = "Cluster Keenam";
                    }
                    else
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            if (jumlah_cluster == 7)
            {
                selisih = selisih / 7;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                b4 = (4 * selisih) + min;
                b5 = (5 * selisih) + min;
                b6 = (6 * selisih) + min;
                win[1] = min;
                win[2] = ((b2 - b1) / 2) + b1;
                win[3] = ((b3 - b2) / 2) + b2;
                win[4] = ((b4 - b3) / 2) + b3;
                win[5] = ((b5 - b4) / 2) + b4;
                win[6] = ((b6 - b5) / 2) + b5;
                win[7] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] <= (win[1] + distance))
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] >= (win[2] - distance) && hasil[a] <= (win[2] + distance))
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] >= (win[3] - distance) && hasil[a] <= (win[3] + distance))
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] >= (win[4] - distance) && hasil[a] <= (win[4] + distance))
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else if (hasil[a] >= (win[5] - distance) && hasil[a] <= (win[5] + distance))
                    {
                        cluster[a] = 5;
                        nama_cluster[a] = "Cluster Kelima";
                    }
                    else if (hasil[a] >= (win[6] - distance) && hasil[a] <= (win[6] + distance))
                    {
                        cluster[a] = 6;
                        nama_cluster[a] = "Cluster Keenam";
                    }
                    else if (hasil[a] >= (win[7] - distance))
                    {
                        cluster[a] = 7;
                        nama_cluster[a] = "Cluster Ketujuh";
                    }
                    else 
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            if (jumlah_cluster == 8)
            {
                selisih = selisih / 8;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                b4 = (4 * selisih) + min;
                b5 = (5 * selisih) + min;
                b6 = (6 * selisih) + min;
                b7 = (7 * selisih) + min;
                win[1] = min;
                win[2] = ((b2 - b1) / 2) + b1;
                win[3] = ((b3 - b2) / 2) + b2;
                win[4] = ((b4 - b3) / 2) + b3;
                win[5] = ((b5 - b4) / 2) + b4;
                win[6] = ((b6 - b5) / 2) + b5;
                win[7] = ((b7 - b6) / 2) + b6;
                win[8] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] <= (win[1] + distance))
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] >= (win[2] - distance) && hasil[a] <= (win[2] + distance))
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] >= (win[3] - distance) && hasil[a] <= (win[3] + distance))
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] >= (win[4] - distance) && hasil[a] <= (win[4] + distance))
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else if (hasil[a] >= (win[5] - distance) && hasil[a] <= (win[5] + distance))
                    {
                        cluster[a] = 5;
                        nama_cluster[a] = "Cluster Kelima";
                    }
                    else if (hasil[a] >= (win[6] - distance) && hasil[a] <= (win[6] + distance))
                    {
                        cluster[a] = 6;
                        nama_cluster[a] = "Cluster Keenam";
                    }
                    else if (hasil[a] >= (win[7] - distance) && hasil[a] <= (win[7] + distance))
                    {
                        cluster[a] = 7;
                        nama_cluster[a] = "Cluster Ketujuh";
                    }
                    else if (hasil[a] >= (win[8] - distance))
                    {
                        cluster[a] = 8;
                        nama_cluster[a] = "Cluster Kedelapan";
                    }
                    else
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            if (jumlah_cluster == 9)
            {
                selisih = selisih / 9;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                b4 = (4 * selisih) + min;
                b5 = (5 * selisih) + min;
                b6 = (6 * selisih) + min;
                b7 = (7 * selisih) + min;
                b8 = (8 * selisih) + min;
                win[1] = min;
                win[2] = ((b2 - b1) / 2) + b1;
                win[3] = ((b3 - b2) / 2) + b2;
                win[4] = ((b4 - b3) / 2) + b3;
                win[5] = ((b5 - b4) / 2) + b4;
                win[6] = ((b6 - b5) / 2) + b5;
                win[7] = ((b7 - b6) / 2) + b6;
                win[8] = ((b8 - b7) / 2) + b7;
                win[9] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] <= (win[1] + distance))
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] >= (win[2] - distance) && hasil[a] <= (win[2] + distance))
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] >= (win[3] - distance) && hasil[a] <= (win[3] + distance))
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] >= (win[4] - distance) && hasil[a] <= (win[4] + distance))
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else if (hasil[a] >= (win[5] - distance) && hasil[a] <= (win[5] + distance))
                    {
                        cluster[a] = 5;
                        nama_cluster[a] = "Cluster Kelima";
                    }
                    else if (hasil[a] >= (win[6] - distance) && hasil[a] <= (win[6] + distance))
                    {
                        cluster[a] = 6;
                        nama_cluster[a] = "Cluster Keenam";
                    }
                    else if (hasil[a] >= (win[7] - distance) && hasil[a] <= (win[7] + distance))
                    {
                        cluster[a] = 7;
                        nama_cluster[a] = "Cluster Ketujuh";
                    }
                    else if (hasil[a] >= (win[8] - distance) && hasil[a] <= (win[8] + distance))
                    {
                        cluster[a] = 8;
                        nama_cluster[a] = "Cluster Kedelapan";
                    }
                    else if (hasil[a] >= (win[9] - distance))
                    {
                        cluster[a] = 9;
                        nama_cluster[a] = "Cluster Kesembilan";
                    }
                    else
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            if (jumlah_cluster == 10)
            {
                selisih = selisih / 10;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                b4 = (4 * selisih) + min;
                b5 = (5 * selisih) + min;
                b6 = (6 * selisih) + min;
                b7 = (7 * selisih) + min;
                b8 = (8 * selisih) + min;
                b9 = (9 * selisih) + min;
                win[1] = min;
                win[2] = ((b2 - b1) / 2) + b1;
                win[3] = ((b3 - b2) / 2) + b2;
                win[4] = ((b4 - b3) / 2) + b3;
                win[5] = ((b5 - b4) / 2) + b4;
                win[6] = ((b6 - b5) / 2) + b5;
                win[7] = ((b7 - b6) / 2) + b6;
                win[8] = ((b8 - b7) / 2) + b7;
                win[9] = ((b8 - b7) / 2) + b7;
                win[10] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] <= (win[1] + distance))
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] >= (win[2] - distance) && hasil[a] <= (win[2] + distance))
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] >= (win[3] - distance) && hasil[a] <= (win[3] + distance))
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] >= (win[4] - distance) && hasil[a] <= (win[4] + distance))
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else if (hasil[a] >= (win[5] - distance) && hasil[a] <= (win[5] + distance))
                    {
                        cluster[a] = 5;
                        nama_cluster[a] = "Cluster Kelima";
                    }
                    else if (hasil[a] >= (win[6] - distance) && hasil[a] <= (win[6] + distance))
                    {
                        cluster[a] = 6;
                        nama_cluster[a] = "Cluster Keenam";
                    }
                    else if (hasil[a] >= (win[7] - distance) && hasil[a] <= (win[7] + distance))
                    {
                        cluster[a] = 7;
                        nama_cluster[a] = "Cluster Ketujuh";
                    }
                    else if (hasil[a] >= (win[8] - distance) && hasil[a] <= (win[8] + distance))
                    {
                        cluster[a] = 8;
                        nama_cluster[a] = "Cluster Kedelapan";
                    }
                    else if (hasil[a] >= (win[9] - distance) && hasil[a] <= (win[9] + distance))
                    {
                        cluster[a] = 9;
                        nama_cluster[a] = "Cluster Kesembilan";
                    }
                    else if (hasil[a] >= (win[10] - distance))
                    {
                        cluster[a] = 10;
                        nama_cluster[a] = "Cluster Kesepuluh";
                    }
                    else
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            for (int a = 0; a < jumlah_cluster; a++)
            {
                dataGridView10.Rows.Add(1);
                dataGridView10.Rows[a].Cells[0].Value = (a + 1).ToString();
                dataGridView10.Rows[a].Cells[1].Value = win[a + 1].ToString();
            }

            for (int a = 0; a < rowCount; a++)
            {
                dataGridView2.Rows[a].Cells[0].Value = dataGridView1.Rows[a].Cells[0].Value;
                dataGridView2.Rows[a].Cells[1].Value = cluster[a].ToString();
                dataGridView2.Rows[a].Cells[2].Value = nama_cluster[a].ToString();
            }

            MessageBox.Show("SOM Eucledian Distance");
        }
        
       public void SOM_Manhattan()
        {
		int rowCount = ((DataTable)this.dataGridView1.DataSource).Rows.Count;
            int columnCount = ((DataTable)this.dataGridView1.DataSource).Columns.Count;
            double[,] c = new double[rowCount, columnCount];
            double[,] d = new double[rowCount, columnCount];
            double[] terbesar = new double[columnCount];
            double[] terkecil = new double[columnCount];
            double[] hasil = new double[rowCount];
            double[] bobot = new double[columnCount];
            double[] bobot_awal = new double[columnCount];
            double[] bobot_temp = new double[columnCount];
            double[] win = new double[11];
            int[] cluster = new int[rowCount];
            string[] nama_cluster=new string[rowCount];
            double MSE = 2;
            double lerning, bil_mse, min, max, selisih, erorr, b1, b2, b3, b4, b5, b6, b7, b8, b9, b10 = 0;
            int iterasi = 1;
            int acak1, acak2, baris_sembarang, distance, jumlah_cluster = 0;
            Random acak = new Random();
            Random acak_baris = new Random();
            //pemindahan data dari datagrid ke array agar mudah diakses dan pancarian nilai max dan min per kolom
            for (int a = 0; a < rowCount; a++)
            {
                for (int b = 1; b < columnCount; b++)
                {
                    c[a, b] = Convert.ToDouble(dataGridView1.Rows[a].Cells[b].Value);
                    d[a, b] = Convert.ToDouble(dataGridView1.Rows[a].Cells[b].Value);
                }

            }

            dataGridView2.Rows.Add(rowCount);
            //pencarian nilai maksimum dan minimum per kolom
            for (int a = 1; a < columnCount; a++)
            {
                terbesar[a] = Math.Max(c[0, a], c[1, a]);
                terkecil[a] = Math.Min(c[0, a], c[1, a]);

                for (int b = 1; b < rowCount; b++)
                {
                    terbesar[a] = Math.Max(terbesar[a], c[b, a]);
                    terkecil[a] = Math.Min(terkecil[a], c[b, a]);
                }

            }
            //normalisasi data
            for (int a = 1; a < columnCount; a++)
            {
                for (int b = 0; b < rowCount; b++)
                {
                    c[b, a] = (c[b, a] - terkecil[a]) / (terbesar[a] - terkecil[a]);
                    //dataGridView2.Rows[b].Cells[a-1].Value = c[b, a].ToString();                         
                }
            }
            acak1 = Convert.ToInt16(comboBox5.SelectedItem.ToString());
            acak2 = Convert.ToInt16(comboBox6.SelectedItem.ToString());
            //pembuatan nilai bobot secara random
            for (int a = 1; a < columnCount; a++)
            {
                bobot[a] = acak.Next(acak1, acak2);
            }

            for (int a = 1; a < columnCount; a++)
            {
                bobot_awal[a] = bobot[a];
            }

            lerning = Convert.ToDouble(comboBox2.SelectedItem.ToString());
            erorr = Convert.ToDouble(comboBox7.SelectedItem.ToString());
            erorr = erorr / 100;
            
            while (MSE > erorr)
            {
                
                baris_sembarang = acak_baris.Next(0, rowCount - 1);
                for (int col = 1; col < columnCount; col++)
                {
                    bobot_temp[col] = bobot[col];
                    //dataGridView8.Rows[iterasi - 1].Cells[col-1].Value = bobot[col].ToString();
                }
                for (int col = 1; col < columnCount; col++)
                {
                    bobot_awal[col] = bobot[col];
                    bobot[col] = bobot[col] + lerning * (c[baris_sembarang, col] - bobot[col]);

                }
                bil_mse = 0;
                for (int col = 1; col < columnCount; col++)
                {
                    bil_mse = bil_mse + (Math.Pow(bobot_temp[col] - bobot[col], 2));
                }
                MSE = Math.Pow(bil_mse, 0.5);
                dataGridView4.Rows.Add(1);
                dataGridView4.Rows[iterasi - 1].Cells[0].Value = iterasi.ToString();
                dataGridView4.Rows[iterasi - 1].Cells[1].Value = lerning.ToString();
                dataGridView5.Rows.Add(1);
                dataGridView5.Rows[iterasi - 1].Cells[0].Value = iterasi.ToString();
                dataGridView5.Rows[iterasi - 1].Cells[1].Value = MSE.ToString();
                iterasi = iterasi + 1;
                lerning = lerning / 2;
            }
            for (int a = 0; a < rowCount; a++)
            {
                for (int b = 1; b < columnCount; b++)
                {
                    c[a, b] = c[a, b] * bobot[b];
                }
            }
            for (int a = 0; a < rowCount; a++)
            {
                for (int b = 1; b < columnCount; b++)
                {
                    d[a, b] = d[a, b] * bobot_awal[b];
                }
            }
            for (int a = 0; a < rowCount; a++)
            {
                hasil[a] = 0;
                for (int b = 1; b < columnCount; b++)
                {
                    hasil[a] = hasil[a] + (c[a, b] - d[a, b]);
                }
                if (hasil[a] < 0)
                {
                    hasil[a] = hasil[a] * -1;
                }
                dataGridView9.Rows.Add(1);
                dataGridView9.Rows[a].Cells[0].Value = hasil[a].ToString();
            }
            max = Math.Max(hasil[0], hasil[1]);
            min = Math.Min(hasil[0], hasil[1]);
            for (int a = 2; a < rowCount; a++)
            {
                max = Math.Max(max, hasil[a]);
                min = Math.Min(min, hasil[a]);
            }
            
            
            distance = Convert.ToInt16(comboBox8.SelectedItem.ToString());
            selisih = max - min;
            jumlah_cluster = Convert.ToInt16(comboBox9.SelectedItem.ToString());
            //MessageBox.Show("max " + max.ToString() + "\n min " + min.ToString() + "\n dist " + distance.ToString() + "\n selisih " + selisih.ToString());

            if (jumlah_cluster == 3)
            {
                selisih = selisih / 3;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                win[1] = min;
                win[2] = ((b2 - b1) / 2) + b1;
                win[3] = max;


                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] <= (win[1] + distance))
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] >= (win[2] - distance) && hasil[a] <= (win[2] + distance))
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] >= (win[3] - distance))
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }
            if (jumlah_cluster == 4)
            {
                selisih = selisih / 4;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                win[1] = min;
                win[2] = ((b2 - b1) / 2) + b1;
                win[3] = ((b3 - b2) / 2) + b2;
                win[4] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] <= (win[1] + distance))
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] >= (win[2] - distance) && hasil[a] <= (win[2] + distance))
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] >= (win[3] - distance) && hasil[a] <= (win[3] + distance))
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] >= (win[4] - distance))
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            if (jumlah_cluster == 5)
            {
                selisih = selisih / 5;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                b4 = (4 * selisih) + min;
                win[1] = min;
                win[2] = ((b2 - b1) / 2) + b1;
                win[3] = ((b3 - b2) / 2) + b2;
                win[4] = ((b4 - b3) / 2) + b3;
                win[5] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] <= (win[1] + distance))
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] >= (win[2] - distance) && hasil[a] <= (win[2] + distance))
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] >= (win[3] - distance) && hasil[a] <= (win[3] + distance))
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] >= (win[4] - distance) && hasil[a] <= (win[4] + distance))
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else if (hasil[a] >= (win[5] - distance))
                    {
                        cluster[a] = 5;
                        nama_cluster[a] = "Cluster Kelima";
                    }
                    else
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            if (jumlah_cluster == 6)
            {
                selisih = selisih / 6;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                b4 = (4 * selisih) + min;
                b5 = (5 * selisih) + min;
                win[1] = min;
                win[2] = ((b2 - b1) / 2) + b1;
                win[3] = ((b3 - b2) / 2) + b2;
                win[4] = ((b4 - b3) / 2) + b3;
                win[5] = ((b5 - b4) / 2) + b4;
                win[6] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] <= (win[1] + distance))
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] >= (win[2] - distance) && hasil[a] <= (win[2] + distance))
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] >= (win[3] - distance) && hasil[a] <= (win[3] + distance))
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] >= (win[4] - distance) && hasil[a] <= (win[4] + distance))
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else if (hasil[a] >= (win[5] - distance) && hasil[a] <= (win[5] + distance))
                    {
                        cluster[a] = 5;
                        nama_cluster[a] = "Cluster Kelima";
                    }
                    else if (hasil[a] >= (win[6] - distance))
                    {
                        cluster[a] = 6;
                        nama_cluster[a] = "Cluster Keenam";
                    }
                    else
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            if (jumlah_cluster == 7)
            {
                selisih = selisih / 7;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                b4 = (4 * selisih) + min;
                b5 = (5 * selisih) + min;
                b6 = (6 * selisih) + min;
                win[1] = min;
                win[2] = ((b2 - b1) / 2) + b1;
                win[3] = ((b3 - b2) / 2) + b2;
                win[4] = ((b4 - b3) / 2) + b3;
                win[5] = ((b5 - b4) / 2) + b4;
                win[6] = ((b6 - b5) / 2) + b5;
                win[7] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] <= (win[1] + distance))
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] >= (win[2] - distance) && hasil[a] <= (win[2] + distance))
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] >= (win[3] - distance) && hasil[a] <= (win[3] + distance))
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] >= (win[4] - distance) && hasil[a] <= (win[4] + distance))
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else if (hasil[a] >= (win[5] - distance) && hasil[a] <= (win[5] + distance))
                    {
                        cluster[a] = 5;
                        nama_cluster[a] = "Cluster Kelima";
                    }
                    else if (hasil[a] >= (win[6] - distance) && hasil[a] <= (win[6] + distance))
                    {
                        cluster[a] = 6;
                        nama_cluster[a] = "Cluster Keenam";
                    }
                    else if (hasil[a] >= (win[7] - distance))
                    {
                        cluster[a] = 7;
                        nama_cluster[a] = "Cluster Ketujuh";
                    }
                    else
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            if (jumlah_cluster == 8)
            {
                selisih = selisih / 8;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                b4 = (4 * selisih) + min;
                b5 = (5 * selisih) + min;
                b6 = (6 * selisih) + min;
                b7 = (7 * selisih) + min;
                win[1] = min;
                win[2] = ((b2 - b1) / 2) + b1;
                win[3] = ((b3 - b2) / 2) + b2;
                win[4] = ((b4 - b3) / 2) + b3;
                win[5] = ((b5 - b4) / 2) + b4;
                win[6] = ((b6 - b5) / 2) + b5;
                win[7] = ((b7 - b6) / 2) + b6;
                win[8] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] <= (win[1] + distance))
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] >= (win[2] - distance) && hasil[a] <= (win[2] + distance))
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] >= (win[3] - distance) && hasil[a] <= (win[3] + distance))
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] >= (win[4] - distance) && hasil[a] <= (win[4] + distance))
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else if (hasil[a] >= (win[5] - distance) && hasil[a] <= (win[5] + distance))
                    {
                        cluster[a] = 5;
                        nama_cluster[a] = "Cluster Kelima";
                    }
                    else if (hasil[a] >= (win[6] - distance) && hasil[a] <= (win[6] + distance))
                    {
                        cluster[a] = 6;
                        nama_cluster[a] = "Cluster Keenam";
                    }
                    else if (hasil[a] >= (win[7] - distance) && hasil[a] <= (win[7] + distance))
                    {
                        cluster[a] = 7;
                        nama_cluster[a] = "Cluster Ketujuh";
                    }
                    else if (hasil[a] >= (win[8] - distance))
                    {
                        cluster[a] = 8;
                        nama_cluster[a] = "Cluster Kedelapan";
                    }
                    else
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            if (jumlah_cluster == 9)
            {
                selisih = selisih / 9;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                b4 = (4 * selisih) + min;
                b5 = (5 * selisih) + min;
                b6 = (6 * selisih) + min;
                b7 = (7 * selisih) + min;
                b8 = (8 * selisih) + min;
                win[1] = min;
                win[2] = ((b2 - b1) / 2) + b1;
                win[3] = ((b3 - b2) / 2) + b2;
                win[4] = ((b4 - b3) / 2) + b3;
                win[5] = ((b5 - b4) / 2) + b4;
                win[6] = ((b6 - b5) / 2) + b5;
                win[7] = ((b7 - b6) / 2) + b6;
                win[8] = ((b8 - b7) / 2) + b7;
                win[9] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] <= (win[1] + distance))
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] >= (win[2] - distance) && hasil[a] <= (win[2] + distance))
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] >= (win[3] - distance) && hasil[a] <= (win[3] + distance))
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] >= (win[4] - distance) && hasil[a] <= (win[4] + distance))
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else if (hasil[a] >= (win[5] - distance) && hasil[a] <= (win[5] + distance))
                    {
                        cluster[a] = 5;
                        nama_cluster[a] = "Cluster Kelima";
                    }
                    else if (hasil[a] >= (win[6] - distance) && hasil[a] <= (win[6] + distance))
                    {
                        cluster[a] = 6;
                        nama_cluster[a] = "Cluster Keenam";
                    }
                    else if (hasil[a] >= (win[7] - distance) && hasil[a] <= (win[7] + distance))
                    {
                        cluster[a] = 7;
                        nama_cluster[a] = "Cluster Ketujuh";
                    }
                    else if (hasil[a] >= (win[8] - distance) && hasil[a] <= (win[8] + distance))
                    {
                        cluster[a] = 8;
                        nama_cluster[a] = "Cluster Kedelapan";
                    }
                    else if (hasil[a] >= (win[9] - distance))
                    {
                        cluster[a] = 9;
                        nama_cluster[a] = "Cluster Kesembilan";
                    }
                    else
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }

            if (jumlah_cluster == 10)
            {
                selisih = selisih / 10;
                b1 = (selisih) + min;
                b2 = (2 * selisih) + min;
                b3 = (3 * selisih) + min;
                b4 = (4 * selisih) + min;
                b5 = (5 * selisih) + min;
                b6 = (6 * selisih) + min;
                b7 = (7 * selisih) + min;
                b8 = (8 * selisih) + min;
                b9 = (9 * selisih) + min;
                win[1] = min;
                win[2] = ((b2 - b1) / 2) + b1;
                win[3] = ((b3 - b2) / 2) + b2;
                win[4] = ((b4 - b3) / 2) + b3;
                win[5] = ((b5 - b4) / 2) + b4;
                win[6] = ((b6 - b5) / 2) + b5;
                win[7] = ((b7 - b6) / 2) + b6;
                win[8] = ((b8 - b7) / 2) + b7;
                win[9] = ((b8 - b7) / 2) + b7;
                win[10] = max;

                for (int a = 0; a < rowCount; a++)
                {
                    if (hasil[a] <= (win[1] + distance))
                    {
                        cluster[a] = 1;
                        nama_cluster[a] = "Cluster Pertama";
                    }
                    else if (hasil[a] >= (win[2] - distance) && hasil[a] <= (win[2] + distance))
                    {
                        cluster[a] = 2;
                        nama_cluster[a] = "Cluster Kedua";
                    }
                    else if (hasil[a] >= (win[3] - distance) && hasil[a] <= (win[3] + distance))
                    {
                        cluster[a] = 3;
                        nama_cluster[a] = "Cluster Ketiga";
                    }
                    else if (hasil[a] >= (win[4] - distance) && hasil[a] <= (win[4] + distance))
                    {
                        cluster[a] = 4;
                        nama_cluster[a] = "Cluster Keempat";
                    }
                    else if (hasil[a] >= (win[5] - distance) && hasil[a] <= (win[5] + distance))
                    {
                        cluster[a] = 5;
                        nama_cluster[a] = "Cluster Kelima";
                    }
                    else if (hasil[a] >= (win[6] - distance) && hasil[a] <= (win[6] + distance))
                    {
                        cluster[a] = 6;
                        nama_cluster[a] = "Cluster Keenam";
                    }
                    else if (hasil[a] >= (win[7] - distance) && hasil[a] <= (win[7] + distance))
                    {
                        cluster[a] = 7;
                        nama_cluster[a] = "Cluster Ketujuh";
                    }
                    else if (hasil[a] >= (win[8] - distance) && hasil[a] <= (win[8] + distance))
                    {
                        cluster[a] = 8;
                        nama_cluster[a] = "Cluster Kedelapan";
                    }
                    else if (hasil[a] >= (win[9] - distance) && hasil[a] <= (win[9] + distance))
                    {
                        cluster[a] = 9;
                        nama_cluster[a] = "Cluster Kesembilan";
                    }
                    else if (hasil[a] >= (win[10] - distance))
                    {
                        cluster[a] = 10;
                        nama_cluster[a] = "Cluster Kesepuluh";
                    }
                    else
                    {
                        cluster[a] = 0;
                        nama_cluster[a] = "Outlier";
                    }
                }
            }
            for (int a = 0; a < jumlah_cluster; a++)
            {
                dataGridView10.Rows.Add(1);
                dataGridView10.Rows[a].Cells[0].Value = (a + 1).ToString();
                dataGridView10.Rows[a].Cells[1].Value = win[a + 1].ToString();
            }

            for (int a = 0; a < rowCount; a++)
            {
                dataGridView2.Rows[a].Cells[0].Value = dataGridView1.Rows[a].Cells[0].Value;
                dataGridView2.Rows[a].Cells[1].Value = cluster[a].ToString();
                dataGridView2.Rows[a].Cells[2].Value = nama_cluster[a].ToString();
            }

            MessageBox.Show("SOM With Manhattan Distance");
		}

       public void insert_propinsi()
       {
           if ((textBox2.Text.Trim().Length == 0))
           {
               MessageBox.Show("Anda harus mengisi nama propinsi dahulu", "Text box tidak boleh kosong", MessageBoxButtons.OK, MessageBoxIcon.Hand);
               return;
           }

           MySqlConnection conn = new MySqlConnection(connectionSQL);
           {
               conn.Open();
               using (MySqlCommand cmd = new MySqlCommand("INSERT INTO propinsi (`propinsi`) VALUES ('" + textBox2.Text + "')", conn))
               {
                 int rows = cmd.ExecuteNonQuery();
               }
               conn.Close();
           }

           textBox2.Clear();
        }

          
       private void button5_Click(object sender, EventArgs e)
       {
           insert_propinsi();
           comboBox4.Items.Clear();
           int i = 0;
           MySqlConnection db = new MySqlConnection(connectionSQL);
           db.Open();
           MySqlCommand showkota = db.CreateCommand();
           string kota = "select propinsi from propinsi";
           
           showkota.CommandText = kota;
           MySqlDataReader viewkota = showkota.ExecuteReader();
           while (viewkota.Read())
           {
               comboBox4.Items.Add(viewkota.GetString(0));

               i = i + 1;
           }

           db.Close();
       }

       private void button4_Click(object sender, EventArgs e)
       {

       }
		
          
   }
}

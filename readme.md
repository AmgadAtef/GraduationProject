using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Text;
using System.Windows.Forms;

using MySql.Data.MySqlClient;
using System.IO;

namespace RTEvents
{
    public partial class afterverifying : Form
    {
        public afterverifying()
        {
            InitializeComponent();

            connectionString = "SERVER=" + server + ";" + "DATABASE=" + database + ";" + "UID=" + uid + ";" + "PASSWORD=" + password + ";";
            connection = new MySqlConnection(connectionString);

            Select();
        }

        void ButtonClick(object sender, EventArgs e)
        {
            Button btn = (Button)sender;
            textBox1.Text = btn.Tag.ToString();
        }

        private MySqlConnection connection;
        private string server = "localhost";
        private string database = "baes";
        private string uid = "root";
        private string password = "";
        
        string connectionString;
        
        /*database stuff*/

        private bool OpenConnection()
        {
            try
            {
                connection.Open();
                return true;
            }
            catch (MySqlException ex)
            {

                switch (ex.Number)
                {
                    case 0:
                        MessageBox.Show("Cannot connect to server.  Contact administrator");
                        break;

                    case 1045:
                        MessageBox.Show("Invalid username/password, please try again");
                        break;
                }
                return false;
            }
        }

        //Close connection
        private bool CloseConnection()
        {
            try
            {
                connection.Close();
                return true;
            }
            catch (MySqlException ex)
            {
                MessageBox.Show(ex.Message);
                return false;
            }
        }

      
        public void Select()
        {
            string query = "SELECT * FROM candidates";

            //Open connection
            if (this.OpenConnection() == true)
            {
                //Create Command
                MySqlCommand cmd = new MySqlCommand(query, connection);
                //Create a data reader and Execute the command
                MySqlDataReader dataReader = cmd.ExecuteReader();
                int i = 1;

                while (dataReader.Read())
                {
                        Button button = new Button();
                        button.Location = new Point(100, 40 * i + 10);
                        button.Tag = dataReader["canid"];
                        button.Text = dataReader["canfullname"] + "";
                        button.Click += new EventHandler(ButtonClick);
                        this.Controls.Add(button);
                        i++;

                        Byte[] bytImage = null;
                        bytImage = (byte[])dataReader["canimage"];

                        if (bytImage != null)
                        {

                            MemoryStream ms = new MemoryStream(bytImage);
                            System.Drawing.Bitmap BMP = new System.Drawing.Bitmap(ms);

                            PictureBox pic = new PictureBox();
                            pic.Location = new Point(150, 40 * i + 10);
                            pic.Image = BMP;
                            pic.SizeMode = PictureBoxSizeMode.Zoom;
                            this.Controls.Add(pic);
                        }
                    
                }

                //close Data Reader
                dataReader.Close();

                //close Connection
                this.CloseConnection();
            }
        }

        public void Insert(String canid)
        {
            string query = "INSERT INTO `votes`(`canid`, `voterid`, `votername`, `voterssn`) VALUES ('"+canid+"','"+RTEvents.RTEventsMain.userid_af+"','"+RTEventsMain.voter_name+"','"+RTEventsMain.voter_ssn+"')";
            
            //open connection
            if (this.OpenConnection() == true)
            {
                //create command and assign the query and connection from the constructor
                MySqlCommand cmd = new MySqlCommand(query, connection);

                //Execute command
                cmd.ExecuteNonQuery();

                //close connection
                this.CloseConnection();
            }
        }


        private void button1_Click(object sender, EventArgs e)
        {
            Insert(textBox1.Text);
            this.Visible = false;
            MessageBox.Show("you have voted successfully", "OK");
        }
    }
}

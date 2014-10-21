
import java.awt.Button;
import java.awt.Color;
import java.awt.Frame;
import java.awt.TextField;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.KeyEvent;
import java.awt.event.KeyListener;
import java.awt.event.WindowEvent;
import java.awt.event.WindowListener;
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.io.RandomAccessFile;
import java.util.Arrays;
import java.util.Comparator;
import javax.swing.SpringLayout;
import java.util.Collections;
import java.util.logging.Level;
import java.util.logging.Logger;
import javax.swing.JOptionPane;

public class AssessmentTracker extends Frame implements ActionListener, WindowListener, KeyListener
{

    private int actualtotalY;
    private int actualtotalX;
    private int totalY = 30;
    private int totalX = 20;
    private TextField[][] fields = new TextField[totalY][totalX];
    private Button clear, save, close, sort, find, search, RAF;
    private String textName = "Answers.csv";
    private TextField searchbox;

    public static void main(String[] args)
    {
        AssessmentTracker frame = new AssessmentTracker();
        frame.run();
    }

    private String topline(int line)
    {
        if (line == 0)
        {
            return "Qns";
        }

        if (line == actualtotalX - 1)
        {
            return "Results";
        }

        //counts the number of questions
        return "Q" + line;
        //}
    }

    private void run()
    {
        setBounds(100, 10, 1090, 790);
        setTitle("Assessment Tracker By Blair Weston");
        this.addWindowListener(this);
        startLayout();
        setVisible(true);
    }

    private void startLayout()
    {
        SpringLayout layout = new SpringLayout();
        setLayout(layout);

        close = new Button("Close");
        add(close);
        close.addActionListener(this);
        layout.putConstraint(SpringLayout.WEST, close, 65, SpringLayout.WEST, this);
        layout.putConstraint(SpringLayout.SOUTH, close, actualtotalY, SpringLayout.SOUTH, this);

        clear = new Button("Clear");
        add(clear);
        clear.addActionListener(this);
        layout.putConstraint(SpringLayout.WEST, clear, 20, SpringLayout.WEST, this);
        layout.putConstraint(SpringLayout.SOUTH, clear, actualtotalY, SpringLayout.SOUTH, this);

        save = new Button("Save");
        add(save);
        save.addActionListener(this);
        layout.putConstraint(SpringLayout.WEST, save, 115, SpringLayout.WEST, this);
        layout.putConstraint(SpringLayout.SOUTH, save, actualtotalY, SpringLayout.SOUTH, this);

        sort = new Button("Sort");
        add(sort);
        sort.addActionListener(this);
        layout.putConstraint(SpringLayout.WEST, sort, 160, SpringLayout.WEST, this);
        layout.putConstraint(SpringLayout.SOUTH, sort, actualtotalY, SpringLayout.SOUTH, this);

        searchbox = new TextField(20);
        add(searchbox);
        searchbox.addActionListener(this);
        layout.putConstraint(SpringLayout.WEST, searchbox, 295, SpringLayout.WEST, this);
        layout.putConstraint(SpringLayout.SOUTH, searchbox, actualtotalY, SpringLayout.SOUTH, this);

        search = new Button("Search");
        add(search);
        search.addActionListener(this);
        layout.putConstraint(SpringLayout.WEST, search, 240, SpringLayout.WEST, this);
        layout.putConstraint(SpringLayout.SOUTH, search, actualtotalY, SpringLayout.SOUTH, this);

        RAF = new Button("RAF");
        add(RAF);
        RAF.addActionListener(this);
        layout.putConstraint(SpringLayout.WEST, RAF, 200, SpringLayout.WEST, this);
        layout.putConstraint(SpringLayout.SOUTH, RAF, actualtotalY, SpringLayout.SOUTH, this);

        //set the field size depending on the csv file
        getActualTotals(textName);

        startData(layout);

        //read in the data once the size of the csv has been calculated
        readFile(textName);

        calculateTotal();
        CalculateAverage();
        CalculateMode();
    }

    private void startData(SpringLayout layout)
    {
        for (int y = 0; y < actualtotalY; y++)
        {
            for (int x = 0; x < actualtotalX; x++)
            {
                if (x == 0)
                {
                    fields[y][x] = new TextField(8);
                } else
                {
                    fields[y][x] = new TextField(4);
                }

                if (y == 0)
                {
                    fields[y][x].setText(topline(x));
                    //fields[y][x].setEditable(false);
                }

                if (x == 0)
                {
                    //fields[y][x].setText(timeLine[y]);
                    //fields[y][x].setEditable(false);
                }

                if (x == actualtotalX)
                {
                    fields[y][x].setBackground(new Color(153, 255, 153));
                    //fields[y][x].setEditable(false);
                }

                if (y > 1 && y < actualtotalY - 1 && x == 0)
                {
                    fields[y][x].setBackground(new Color(255, 255, 153));
                    //fields[y][x].setEditable(false);
                }

                if (y < 1 && x < actualtotalX)
                {
                    fields[y][x].setBackground(new Color(0, 128, 255));
                    //fields[y][x].setEditable(false);
                }

                if (y == actualtotalY - 1 && x < actualtotalX || y > 1 && x == actualtotalX - 1)
                {
                    fields[y][x].setBackground(new Color(145, 255, 145));
                    //fields[y][x].setEditable(false);
                }

                if (y == actualtotalY - 1 && x == actualtotalX - 1)
                {
                    fields[y][x].setBackground(new Color(255, 255, 153));
                    //fields[y][x].setEditable(false);
                }

                add(fields[y][x]);
                fields[y][x].addKeyListener(this);

                layout.putConstraint(SpringLayout.WEST, fields[y][x], x * 90 + 10, SpringLayout.WEST, this);
                layout.putConstraint(SpringLayout.NORTH, fields[y][x], y * 30 + 10, SpringLayout.NORTH, this);

            }
        }
    }

    //clears all the text boxes as well as resets the colour if a previous search was on
    private void clearData()
    {
        ReColour();
        for (int y = 1; y < actualtotalY; y++)
        {
            for (int x = 0; x < actualtotalX; x++)
            {
                if (y == 1 && x == 0)
                {
                    continue;
                }
                if (y == actualtotalY - 1 && x == 0)
                {
                    continue;
                }
                fields[y][x].setText("");

            }
        }
    }

    @Override
    public void actionPerformed(ActionEvent e)
    {
        if (e.getSource() == close)
        {
            System.exit(0);
        }

        if (e.getSource() == clear)
        {
            clearData();
        }

        if (e.getSource() == save)
        {
            writeFile(textName);
        }

        if (e.getSource() == sort)
        {
            System.out.println("Sort Button");
            SortNames();
        }

        if (e.getSource() == search)
        {
            SearchNames();
        }

        if (e.getSource() == RAF)
        {
            try
            {
                RAF();
            } catch (FileNotFoundException ex)
            {
                Logger.getLogger(AssessmentTracker.class.getName()).log(Level.SEVERE, null, ex);
            }
        }

    }

    @Override
    public void windowOpened(WindowEvent e)
    {

    }

    @Override
    public void windowClosed(WindowEvent e)
    {
        System.exit(0);
    }

    @Override
    public void windowIconified(WindowEvent e)
    {

    }

    @Override
    public void windowDeiconified(WindowEvent e)
    {

    }

    @Override
    public void windowActivated(WindowEvent e)
    {

    }

    @Override
    public void windowDeactivated(WindowEvent e)
    {

    }

    @Override
    public void keyTyped(KeyEvent e)
    {

    }

    @Override
    public void keyPressed(KeyEvent e)
    {

    }

    @Override
    public void keyReleased(KeyEvent e)
    {
        //calculateTotal();
    }

    private void tryParse(int x, int y)
    {
        try
        {
            Integer.parseInt(fields[y][x].getText());
        } catch (NumberFormatException e)
        {
            fields[y][x].setText("0");
        }
    }

    //go through the array and calculate student score using there score based on the answers
    private void calculateTotal()
    {
        int count = 0;

        for (int y = 2; y < actualtotalY - 1; y++)
        {

            for (int x = 1; x < actualtotalX - 1; x++)
            {

                if (fields[1][x].getText().equals(fields[y][x].getText()))
                {
                    count++;
                }
            }
            fields[y][actualtotalX - 1].setText(count + "");
            count = 0;

        }
    }

    //calculate all the scores and divide by the amount of students 
    //then displays the avg in the bottom right hand box
    private void CalculateAverage()
    {
        int students = actualtotalY - 3;
        float total = 0;

        for (int y = 2; y < actualtotalY - 1; y++)
        {
            total += Integer.parseInt(fields[y][actualtotalX - 1].getText());
        }
        float avg = total / students;
        fields[actualtotalY - 1][actualtotalX - 1].setText(avg + "");
    }

    //going down each row and then displays the most common answer for each question
    private void CalculateMode()
    {
        //start in the first answer field
        for (int x = 1; x < actualtotalX - 1; x++)
        {
            //set 4 elements for the array A,B,C,D
            int[] modes = new int[4];

            //starts in the first students row
            for (int y = 2; y < actualtotalY - 1; y++)
            {
                //set the area to collect the data from
                switch (fields[y][x].getText())
                {
                    // if text = A then adds 1 to the modes counter
                    // this is repeated for each Case depending on the letter
                    case "A":
                        modes[0]++;
                        break;

                    case "B":
                        modes[1]++;
                        break;

                    case "C":
                        modes[2]++;
                        break;

                    case "D":
                        modes[3]++;
                        break;
                }
            }
            //set 2 counters for the mode and max count
            int mode = 0;
            int max = 0;

            //while i is less then the 
            for (int i = 0; i < modes.length; i++)
            {
                //goes through the modes array if mode i is larger then current value then set to max
                if (modes[i] > max)
                {
                    //max now = the mode in the modes array that has the largest number
                    max = modes[i];
                    //mode is now set to = the number in the modes array that has largest number
                    mode = i;

                    //search for the number in the array that matches the mode number and convert to the 
                    //relevent text
                    switch (mode)
                    {
                        case 0:
                            fields[actualtotalY - 1][x].setText("A");
                            break;

                        case 1:
                            fields[actualtotalY - 1][x].setText("B");
                            break;

                        case 2:
                            fields[actualtotalY - 1][x].setText("C");
                            break;

                        case 3:
                            fields[actualtotalY - 1][x].setText("D");
                            break;

                    }
                }
            }
        }
    }

    // set string student to convert array method
    public void SortNames()
    {
        String[][] Student = ConvertArray();
        System.out.println(Student.length);
        sortArray(Student);
        updateArray(Student);
    }

    private String[][] ConvertArray()
    {
        String[][] Student = new String[actualtotalY][actualtotalX];
        for (int y = 0; y < actualtotalY; y++)
        {
            for (int x = 0; x < actualtotalX; x++)
            {
                Student[y][x] = fields[y][x].getText();
            }
        }
        return Student;
    }
    
    private void sortArray(String[][] data)
    {
        // sort the array data
        Arrays.sort(data, new Comparator<String[]>()
        {
            @Override
            public int compare(final String[] first, final String[] second)
            {
                //compare the first element in the array with the second
                //this continues throughout the entire array
                if(validate(first[0])&& validate(second[0]))
                {
                    final String element1 = first[0];
                    final String element2 = second[0];
                    return element1.compareTo(element2);
                }
                else
                {
                    return 0;
                }
            }
        });
    }
    //fields in the array that do not need to be sorted
    private boolean validate(String val)
    {
        //when set to false will not sort
        boolean result = false;
        switch(val)
        {
            case "Qns":
                result = false;
                break;
                
            case "Answers":
                result = false;
                break;
                
            case "MODE:":
                result = false;
                break;
        
                // if section of array is not one of the above cases
                // then set to true and sort
            default:
                result = true;
                break;
        }
        return result;
    }
    
    //writes to the screen the current sorted array
    private void updateArray(String [][] sort)
    {
        for (int y = 0; y < actualtotalY; y++)
        {
            for (int x = 0; x < actualtotalX; x++)
            {
                fields[y][x].setText(sort[y][x]);
            }
        }
    }
    
    private void SearchNames()
    {
        //set the default colour layout back to what it was
        ReColour();

        for (int y = 2; y < actualtotalY - 1; y++)
        {
            //searches for matching text and then highlights the row
            if (fields[y][0].getText().equals(searchbox.getText()))
            {
                for (int x = 0; x < actualtotalX; x++)
                {
                    fields[y][x].setBackground(Color.LIGHT_GRAY);
                }

            }
        }
    }

    // ReColour method sets the screen back to default colours to clear any previous searches
    private void ReColour()
    {
        for (int y = 0; y < actualtotalY; y++)
        {
            for (int x = 0; x < actualtotalX; x++)
            {
                if (y > 0  && x < actualtotalX - 1)
                {
                    fields[y][x].setBackground(Color.WHITE);
                }

                if (y > 1 && y < actualtotalY - 1 && x == 0)
                {
                    fields[y][x].setBackground(new Color(255, 255, 153));
                    fields[y][x].setEditable(false);
                }

                if (y == actualtotalY - 1 && x < actualtotalX || y > 1 && x == actualtotalX - 1)
                {
                    fields[y][x].setBackground(new Color(145, 255, 145));
                    fields[y][x].setEditable(false);
                }
                
                if (y == actualtotalY -1 && x == actualtotalX -1)
                {
                    fields[y][x].setBackground(new Color(255, 255, 153));
                }

            }
        }
    }

    private void RAF() throws FileNotFoundException
    {

        try
        {
            // create a new RandomAccessFile with filename RAF
            RandomAccessFile raf = new RandomAccessFile("RAF.txt", "rw");

            for (int y = 1; y < actualtotalY - 1; y++)
            {
                for (int x = 0; x < actualtotalX - 1; x++)
                {
                    raf.writeUTF(fields[y][x].getText() + ",");
                }

            }
        } catch (IOException ex)
        {
            ex.printStackTrace();
        }
        //alert the user that RAF file has been created
        JOptionPane.showMessageDialog(null, "RAF File has been saved");
    }

    @Override
    public void windowClosing(WindowEvent e)
    {
        System.exit(0);
    }

    //this method is called before reading the data so as the program can setup how big the fields need to be
    // before reading in the information in the csv file
    private void getActualTotals(String fileName)
    {
        try
        {
            BufferedReader br;
            br = new BufferedReader(new FileReader(fileName));
            int yCount = 2;
            String line;

            while (((line = br.readLine()) != null))
            {
                String temp[] = line.split(",");
                yCount++;

                actualtotalY = yCount;
                actualtotalX = temp.length + 1;
                System.out.println("" + actualtotalY);
            }
        } catch (Exception e)
        {
            System.out.println("Error: " + e.toString());
        }
        readFile(textName);
    }

    private void readFile(String fileName)
    {
        try
        {
            BufferedReader br;
            br = new BufferedReader(new FileReader(fileName));
            int count = 0;

            for (int y = 0; y < actualtotalY - 1; y++)
            {
                String temp[] = br.readLine().split(",");

                for (int x = 0; x < actualtotalX - 1; x++)
                {
                    fields[y + 1][x].setText(temp[x]);
                    fields[actualtotalY - 1][0].setText("MODE:");
                    count++;

                }
            }

        } catch (Exception e)
        {

        }

    }

    public void writeFile(String fileName)
    {
        try
        {
            // Create file and start streams
            BufferedWriter out;
            out = new BufferedWriter(new FileWriter(fileName));
            for (int y = 1; y < actualtotalY - 1; y++)
            {
                for (int x = 0; x < actualtotalX - 1; x++)
                {
                    out.write(fields[y][x].getText() + ",");

                }
                out.newLine();
            }
            out.close(); // close stream //if(y !=0 ) 
        } catch (IOException e)
        {
            System.err.println("Error: " + e.getMessage()); // print message on error
        }
    }
}


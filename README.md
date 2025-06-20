package com.comlinkusa.financeoneui.client;

import com.comlinkusa.financeoneui.common.*;
import com.comlinkusa.financeoneui.guilib.BasePanel;
import com.comlinkusa.financeoneui.logger.ComlinkLogger;
import com.comlinkusa.financeoneui.server.CommSvrImpl;
import com.comlinkusa.financeoneui.session.SessionService;

import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.FocusEvent;
import java.awt.event.FocusListener;
import java.io.File;
import java.io.IOException;
import java.lang.reflect.Field;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.util.ArrayList;
import java.util.logging.Logger;

/*Start of IR No 18100012*/
/*End of IR No 18100012*/

public class FinanceOneLogon extends JFrame
{
  private static File SETTINGS_FILE;
  private ComlinkLogger financeOneLogger;
  private ApplicationSettings appSettings;
  private CommSvrImpl server;
  private JLabel userIDLabel;
  private JTextField userIDField;
  private JLabel passwordLabel;
  private JPasswordField passwordField;
  private JButton okButton;
  private JButton cancelButton;
  private static String vhost;
  private static int vport = 0;
  private static String vuser = "";
  private static String vpass = "";
  private boolean isError = true;
  private boolean messagewindow = true;
  private boolean messagereply;
  private static String errorMessage = "";
  static String p="";
  private static int q;

  static Logger logger= Logger.getLogger(FinanceOneLogon.class.getName());
  
  public FinanceOneLogon() {
    super("F1: Login");
    this.financeOneLogger = ComlinkLogger.getInstance();
    this.appSettings = ApplicationSettings.getInstance();
    this.userIDLabel = null;
    this.userIDField = null;
    this.passwordLabel = null;
    this.passwordField = null;
    this.okButton = null;
    this.cancelButton = null;
    try
    {
      UIManager.setLookAndFeel("com.sun.java.swing.plaf.windows.WindowsLookAndFeel");

      initComponents();
    }
    catch (FinanceOneException financeoneexception)
    {
      if (financeoneexception.getMessage() != "")
      {
        JOptionPane.showMessageDialog(null, financeoneexception.getMessage(), "Unix Logon: Error Message", 0);
        System.exit(0);
      }
      else if (financeoneexception.getMessage() == "")
      {
        JOptionPane.showMessageDialog(null, "Already in progress", "Unix Logon: Error Message", 0);
        System.exit(0);
      }

    }
    //*****************START OF CR 20100050******************
    catch (ClassNotFoundException | InstantiationException | IllegalAccessException | UnsupportedLookAndFeelException exception)
    {
      this.financeOneLogger.log("com.comlinkusa.financeoneui.guide.FinanceOneLogon", "constructor", "Unable to set the application's look and feel:\n" + exception.getMessage());
    }
    //*****************END OF CR 20100050******************
    this.financeOneLogger.log("com.comlinkusa.financeoneui.client.FinanceOneLogon", "Constructor", "new FinanceOneLogon instance created");
    addWindowListener(new WindowMonitor());
    pack();
    setSize(800, 585);
    setResizable(false);
  }

  private void initComponents()
    throws FinanceOneException
  {
    try
    {
      this.appSettings.resetApplicationSettings(SETTINGS_FILE);
    }
    catch (IOException ioexception)
    {
      this.financeOneLogger.log("com.comlinkusa.financeoneui.client.FinanceOneLogon", "initComponents", "Unable to reset application settings in file " + SETTINGS_FILE);
      throw new FinanceOneException(ioexception.toString(), 3);
    }
    try
    {
      this.server = new CommSvrImpl();
    }
    catch (IOException ioexception1)
    {
      this.financeOneLogger.log("com.comlinkusa.financeoneui.client.FinanceOneLogon", "initComponents", "Some problem constructing the CommSvrImpl object: " + ioexception1);
      throw new FinanceOneException(ioexception1.toString(), 3);
    }
    /*Start of IR No 17020017*/
    RuntimeConstants runtimeconstants = new RuntimeConstants();
    ArrayList Arrayhostreturn = new ArrayList();
    ArrayList Arrayportreturn = new ArrayList();
    try {
      Arrayhostreturn = runtimeconstants.getComlinkHost1();
    
      Arrayportreturn = runtimeconstants.getComlinkPort1();


      this.server.getPool().setSize(1);
      this.server.getPool().setMax(1);
      this.server.getPool().setUsername(runtimeconstants.getF1User());

      this.server.getPool().setPassword(runtimeconstants.getF1Password());

      Registry registry = LocateRegistry.createRegistry(7001);

      int length = Arrayhostreturn.size();

      boolean flag = false;

      if (this.isError)
      {
        new Thread(new Runnable()
        {
          public void run()
          {
            JLabel errorFields = new JLabel("<HTML><FONT COLOR = Blue>    Please Wait Connecting to server...<br>Do not close this window      </FONT></HTML>");

            int n = JOptionPane.showOptionDialog(null, errorFields, "Alert", -1, 1, null, new Object[0], null);
            int m = 0;
          }
        }).start();

        flag = true;
       
      }
      String s = null;
      int portreturn = 0;
      for (int i = 0; i < length; ++i)
      {
        try
        {
          if (this.isError)
          {
             s = (String)Arrayhostreturn.get(i);

             portreturn = Integer.parseInt((String)Arrayportreturn.get(i));

            this.server.getPool().setHost(s);

            this.server.getPool().setPort(portreturn);


            this.server.getPool().initializePool();
            registry.rebind("CommServer", this.server);

            this.isError = false;
          }

          if (!this.isError)
          {
            JOptionPane.getRootFrame().dispose();
          }

         
           p = s;
           q=portreturn;
         
        }
        //*****************START OF CR 20100050******************
        catch (HeadlessException | NumberFormatException | RemoteException exception)
        {
          this.financeOneLogger.log("com.comlinkusa.financeoneui.client.FinanceOneLogon", "actionPerformed", exception.getMessage());

          errorMessage = exception.getMessage();
         
          this.isError = true;
        }
        //*****************END OF CR 20100050******************

      }

    }
    catch (IOException ioexception2)
    {
      this.financeOneLogger.log("com.comlinkusa.financeoneui.client.FinanceOneLogon", "initComponents", "Some problem from the server object ");
    }
    //*****************START OF CR 20100050******************
    catch (HeadlessException | NumberFormatException e) {
		/*Start of IR No 18100012*/
      //this.financeOneLogger.log(e.printStackTrace());
      logger.info("***** In HeadlessException | NumberFormatException*****");
		//e.printStackTrace();
	  /*END of IR No 18100012*/
    }
    //*****************END OF CR 20100050******************
    if (this.isError)
    {
      
      throw new FinanceOneException(errorMessage, 1);
    }
    /*End of IR No 17020017*/
    Container container = getContentPane();
    container.setLayout(new BorderLayout());
    JLabel jlabel = new JLabel(Utils.COMLINK);
    jlabel.setBorder(BorderFactory.createLineBorder(Color.BLACK));
    container.add(jlabel, "West");
    JPanel jpanel = new JPanel(new BorderLayout());
    jpanel.setBorder(BorderFactory.createEtchedBorder());
    this.userIDLabel = new JLabel("USER ID", 4);
    this.userIDField = new LimitedTextField(15, false, true);
    this.passwordLabel = new JLabel("PASSWORD", 4);
    this.passwordField = new JPasswordField();
    this.okButton = new JButton("OK");
    this.cancelButton = new JButton("Cancel");
    this.passwordField.addFocusListener(new FocusListener()
    {
      public void focusGained(FocusEvent focusevent)
      {
        FinanceOneLogon.this.passwordField.selectAll();
      }

      public void focusLost(FocusEvent focusevent)
      {
        FinanceOneLogon.this.passwordField.select(0, 0);
      }
    });
    JPanel jpanel1 = new JPanel(new GridBagLayout());
    jpanel1.setAlignmentX(17.5F);
    Utils.addParameterRow(jpanel1, this.userIDLabel, this.userIDField);
    Utils.addParameterRow(jpanel1, this.passwordLabel, this.passwordField);
    JPanel jpanel2 = new JPanel(new FlowLayout(0));
    /*Start of IR No 19060093*/
    JLabel jlabel1 = new JLabel("<html>Finance One Login <br><br><font size='3'>APM code : core_Info_220</font></html>", 2);
	/*END of IR No 19060093*/
    jlabel1.setFont(new Font("Arial", 1, 18));
    jlabel1.setAlignmentX(0.0F);
    jpanel2.add(jlabel1);
    jpanel2.add(Box.createRigidArea(new Dimension(400, 12)));
    jpanel2.add(Box.createRigidArea(new Dimension(0, 100)));
    jpanel2.add(jpanel1);
    this.okButton.addActionListener(new ActionListener()
    {
      public void actionPerformed(ActionEvent actionevent)
      {
        EventQueue.invokeLater(new Runnable()
        {
          public void run()
          {
            FinanceOneLogon.this.setCursor(Cursor.getPredefinedCursor(3));
            String s = null;
            try
            {
              char[] s1 = (FinanceOneLogon.this.passwordField.getPassword());
              if ((FinanceOneLogon.this.userIDField.getText() == null) || (FinanceOneLogon.this.userIDField.getText().length() <= 0)) {
                s = "Login failed: Please enter a valid user id";
              }
              else if ((s1 == null) || (s1.length <= 0))
                s = "Login failed: Please enter a valid password";
              if (s != null) {
                //break label418;
              }
              StringBuffer stringbuffer = new StringBuffer();
              BasePanel basepanel = new BasePanel();
              stringbuffer.append(basepanel.addPadding("LOGON", 8));
              stringbuffer.append(basepanel.addPadding(FinanceOneLogon.this.userIDField.getText().trim(), 8));
              stringbuffer.append(basepanel.addPadding("1", 3));
              stringbuffer.append(basepanel.addPadding(FinanceOneLogon.this.userIDField.getText().trim(), 8));
              stringbuffer.append(basepanel.addPadding(s1.toString(), 8));
              String s2 = FinanceOneLogon.this.server.sendRequest(stringbuffer.toString());
              s1=null;
			      /*Start of IR No 18100012*/
              //System.out.println("----s2 while login attemp ---- "+s2);
              /*End of IR No 18100012*/
              if (s2.indexOf("*DB-ERR*") != -1) {
                s = "Login failed: Please enter a valid user id and password";
              }
              else if (s2.indexOf("** ERROR ON -") != -1)
              {
                int i = s2.indexOf("** ERROR");
                int j = i + 60;
                s = s2.substring(i + 2, j).trim() + "\n" + s2.substring(j, j + 60).trim();
              }
              else {
                String s3 = s2.substring(0, 8).trim();
                FinanceOne financeone = new FinanceOne(true);
                financeone.setUserID(FinanceOneLogon.this.userIDField.getText());
                financeone.setTransactionCode(s3);
                financeone.setWindowID(FinanceOne.getAvailableWindowID());
                financeone.addWindowToMap(s3, financeone);
                financeone.show();
                FinanceOneLogon.this.setVisible(false);
                FinanceOneLogon.this.dispose();
              }

            }
            catch (RemoteException exception1)
            {
              s = exception1.getMessage();
              FinanceOneLogon.this.financeOneLogger.log("com.comlinkusa.financeoneui.client.FinanceOneLogon", "actionPerformed", s);
            }
            finally
            {
              if (s != null)
                JOptionPane.showMessageDialog(null, s, "F1 Logon: Error Message", 0);
              FinanceOneLogon.this.setCursor(Cursor.getPredefinedCursor(0));
            }
          }
        });
      }
    });
    JRootPane jrootpane = getRootPane();
    jrootpane.setDefaultButton(this.okButton);
    this.cancelButton.addActionListener(new ActionListener()
    {
      public void actionPerformed(ActionEvent actionevent)
      {
        FinanceOneLogon.this.setVisible(false);
        FinanceOneLogon.this.dispose();
        System.exit(0);
      }
    });
    JLabel jlabel2 = new JLabel(Utils.CLIENT);
    jlabel2.setPreferredSize(new Dimension(480, 40));
    jlabel2.setOpaque(true);
    JPanel jpanel3 = new JPanel(new FlowLayout(2));
    this.okButton.setPreferredSize(new Dimension(73, 27));
    this.cancelButton.setPreferredSize(new Dimension(73, 27));
    jpanel3.add(jlabel2);
    jpanel3.add(this.okButton);
    jpanel3.add(this.cancelButton);
    jpanel.add(jpanel2, "Center");
    jpanel.add(jpanel3, "South");
    container.add(jpanel, "Center");

    Dimension dimension = Toolkit.getDefaultToolkit().getScreenSize();
    setLocation((dimension.width - 800) / 2, (dimension.height - 585) / 2);
  }



  public static String getTheDdfName()
  {
    String s = null;
    String s1 = "com.comlinkusa.financeoneui.client.DdfFileNameClass";
    String s2 = "ddfFileName";
    try
    {
      Class class1 = Class.forName(s1);
      Field field = class1.getDeclaredField(s2);
      s = (String)field.get(null);
    }
    catch (ClassNotFoundException classnotfoundexception)
    {
      JFrame jframe = new JFrame();
      JOptionPane.showMessageDialog(jframe, "Cannot find the class " + s1);
      System.exit(0);
    }
    catch (IllegalAccessException illegalaccessexception)
    {
      JFrame jframe1 = new JFrame();
      JOptionPane.showMessageDialog(jframe1, "Illegal access exception:" + illegalaccessexception);
      System.exit(0);
    }
    catch (NoSuchFieldException nosuchfieldexception)
    {
      JFrame jframe2 = new JFrame();
      JOptionPane.showMessageDialog(jframe2, "Cannot find the field " + s2 + " in class " + s1);
      System.exit(0);
    }
    return s;
  }
  
  //Fix Starts -20/06/2025
  public static void terminateAppGracefully(){
		for(Window window : window.getWindows()){
				window.dispose();
		}
	Runtime.getRuntime.exit(0);
  }
  //Fix Ends

  public static void main(String[] args) throws Exception
  {
    if ((args != null) && (args.length > 0))
    {
      for (int i = 0; i < args.length; ++i) {
        if (!args[i].equalsIgnoreCase("-version"))
          continue;
        RuntimeConstants runtimeconstants = new RuntimeConstants();
        ArrayList Arrayhostreturn1 = new ArrayList();
        ArrayList Arrayportreturn1 = new ArrayList();
        Arrayhostreturn1 = runtimeconstants.getComlinkHost1();
        
        Arrayportreturn1 = runtimeconstants.getComlinkPort1();
       
        int length1 = Arrayhostreturn1.size();
        for (int k = 0; k < length1; ++k)
        {
          String s2 = (String)Arrayhostreturn1.get(k);
          
          int portreturn = Integer.parseInt((String)Arrayportreturn1.get(k));
         
          vhost = s2;
          vport = portreturn;
          vuser = runtimeconstants.getF1User();
          vpass = runtimeconstants.getF1Password();
          JFrame jframe = new JFrame();
          JOptionPane.showMessageDialog(jframe, "IP      = " + vhost + "\nPort    = " + vport + "\nUser    = " + vuser + "\nPassword= " + vpass);
          System.exit(0);
        }

      }

    }
    
    /*Start of IR No 18100012*/
	if(SessionService.getAppTime()==0) {
	System.exit(0);
	} else {
		SessionService.sessioOutTimeInitializer();
	}
	 /*End of IR No 18100012*/

    String s = getTheDdfName();
    if (s != null)
    {
      SETTINGS_FILE = new File(s);
    }
    else {
      JOptionPane.showMessageDialog(null, "The ddf file name did not get defined by guide\n", "F1 Logon: Error Message", 0);
      System.exit(0);
    }
    if (Utils.verify())
    {
      new FinanceOneLogon().show();
    }
    else {
      JOptionPane.showMessageDialog(null, "Your license has expired. Please contact Born.", "F1 Logon: Error Message", 0);
      System.exit(0);
    }
  }
}

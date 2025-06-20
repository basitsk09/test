package com.comlinkusa.financeoneui.session;
import java.io.InputStream;

import javax.swing.JFrame;
import javax.swing.JOptionPane;

import com.comlinkusa.financeoneui.client.A515E;
import org.apache.log4j.Logger;

import com.comlinkusa.financeoneui.client.FinanceOne;
//new imports start
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
//new imports end


/**
 * 
 * TODO
 *
 */
public class AppSession extends JFrame{
	
	int p;
	public static int a = 0;
	static FinanceOne f1Obj = null;

	static Logger logger= Logger.getLogger(AppSession.class.getName());
	private final ExecutorService executor = Executors.newSingleThreadExecutor();

	//static Logger logger = Logger.getLogger(AppSession.class.getName());
	/*static String str;
	static long msgTime;
	public static int appTime;*/
	static InputStream input = null;
	
	public AppSession() {
		timerEvent();

	}
	
	/*private static final String TIME_PROPERTIES_FILE_NOT_FOUND = "TimeProperties file not found!!!";
	private static final String MESSAGE = "message";
	private static final String MESSAGE_TIME = "messageTime";
	private static final String IDLE_TIME = "idleTime";
	private static final String YOUT_KEY = "AVX2oX8TYBCTCSXXOO12";

	private static Properties getProp() {
		Properties properties = null;
		try {
			logger.i("in properties initialization");
			properties = new Properties();
			input = new FileInputStream("TimeProperties.properties");
			properties.load(input);
			if(properties.getProperty(IDLE_TIME)==null||properties.getProperty(MESSAGE)==null||properties.getProperty(MESSAGE_TIME)==null) {
				JOptionPane.showMessageDialog(null, "TimeProperties file is tempered!!!");
				System.out.println("exiting Application............................");
				System.exit(0);
			}
		} catch (IOException e) {
		
			JOptionPane.showMessageDialog(null, TIME_PROPERTIES_FILE_NOT_FOUND, "Error: Time Properties file", JOptionPane.ERROR_MESSAGE);
			//e.printStackTrace();
			System.out.println("please insert file");
		}
		return properties;
	}

	public static int getAppTime() {
		String strToDecrypt = getProp().getProperty(IDLE_TIME);
		if(strToDecrypt!=null) {
		String decryptedString = Decrypt_Util.Decrypt(strToDecrypt, YOUT_KEY);
		System.out.println("decrypted String:" + decryptedString);	
		if(decryptedString!=null) {
		appTime = Integer.parseInt(decryptedString);
		
			System.out.println("AppTime: "+appTime);
			return appTime;
		} else
			System.exit(0);
		return 0;
		} else {
			System.out.println("please place Time Properties file in the same folder as the jar file: 0");
			return 0;
		}

	}

	public static void sessioOutTimeInitializer() {
		appTime = getAppTime();
		String no = getProp().getProperty(MESSAGE_TIME);
		System.out.println("Decrypted message time:" + " " + Decrypt_Util.Decrypt(no, YOUT_KEY));
		msgTime = Long.parseLong(Decrypt_Util.Decrypt(no, YOUT_KEY));
		String strToDecrypt = getProp().getProperty(MESSAGE);

		str = Decrypt_Util.Decrypt(strToDecrypt, YOUT_KEY);
		System.out.println("decrypted message:" + " " + str);
	}
*/
	/**
	 * This method is responsible for triggering the warning message
	 * 
	 */
	
	public void closeAllProcess() {
		
		String[] keySet = (String[]) f1Obj.getWindowMap().keySet().toArray(new String[0]);
		
		for(String  key :keySet) {
			//System.out.println("--- kry--- "+key);
			//System.out.println("--val -- "+ f1Obj.getWindowMap().get(key));
			FinanceOne f1= (FinanceOne)f1Obj.getWindowMap().get(key);
			
			//System.out.println("sub-Frame flag:: "+f1.isParent());
			
			if(f1.isParent()) {
				
				f1.setIsAutoExit(false);
				f1.processCloseRequest(f1.isParent(), f1.isAutoExit());
			}
			
		}
		//System.out.println("Main frame flag:: "+f1Obj.isParent());
		
	}
	
	public void timerEvent() {

		SessionService.sessioOutTimeInitializer();
		f1Obj = new FinanceOne(true);
		
		Thread t = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					//logger.debug("AppSession:: ************************ in try  thread started" + a++);
					Thread.sleep(SessionUtil.msgTime);
				} catch (InterruptedException e2) {
					logger.info("***** In InterruptedException *****");
				}
				JOptionPane.getRootFrame().dispose();
			}
		});
		t.start();
		p = JOptionPane.showConfirmDialog(null, SessionUtil.str, SessionConstant.getDialougetitle(), JOptionPane.YES_NO_OPTION);
		//logger.debug("AppSession:: thread t started");
		if (p == 0) {
			//logger.debug("AppSession:: dispose");
			//f1Obj.sessionCloseFunction(f1Obj.isParent());
			//System.out.println("in 1st if:::: p=0");
			closeAllProcess();
			//f1Obj.sessionCloseFunction();
			//System.exit(0);

		} else if (p == 1) {
			//logger.debug("AppSession:: in no");
			//System.out.println("in else if:::: ");
			a = 0;
			f1Obj.menuFlag = false;
			f1Obj.sessionOut.restart();
		} 
		else if (p == -1) {
			//f1Obj.sessionCloseFunction(f1Obj.isParent());
			//System.out.println("in 2nd if:::: p=-1");
			closeAllProcess();
			//f1Obj.sessionCloseFunction();
			//System.exit(0);
		} else {
			//System.out.println("reaching else!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!");
			f1Obj.menuFlag = false;
			f1Obj.initComponents();


		}

	}

}

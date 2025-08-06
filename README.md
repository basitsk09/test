package com.comlinkusa.financeoneui.session;
import java.io.InputStream;
import javax.swing.JFrame;
import javax.swing.JOptionPane;
import com.comlinkusa.financeoneui.client.A515E;
import org.apache.log4j.Logger;
import com.comlinkusa.financeoneui.client.FinanceOne;

// =================================================================
// FIX START: Replaced concurrent imports with the Swing Timer.
// =================================================================
import javax.swing.Timer;
import java.awt.event.ActionListener;
import java.awt.event.ActionEvent;
// =================================================================
// FIX END
// =================================================================


/**
 * * TODO
 *
 */
public class AppSession extends JFrame{
	
	int p;
	public static int a = 0;
	static FinanceOne f1Obj = null;

	static Logger logger= Logger.getLogger(AppSession.class.getName());
	
    /*
     * =================================================================
     * OLD CODE START: ExecutorService is not needed for this task.
     * A Swing Timer is the correct and simpler tool.
     * =================================================================
     */
	/*
	//new change start
	private final ExecutorService executor = Executors.newSingleThreadExecutor();
    //new change end
    */
    /*
     * =================================================================
     * OLD CODE END
     * =================================================================
     */

	static InputStream input = null;
	
	public AppSession() {
		timerEvent();

        /*
         * =================================================================
         * OLD CODE START: This WindowListener was syntactically incorrect
         * (outside a method) and is no longer needed since we are not
         * manually managing an ExecutorService.
         * =================================================================
         */
        /*
        addWindowListener(new java.awt.event.WindowAdapter() {
            @Override
            public void windowClosing(java.awt.event.WindowEvent e) {
                shutdownExecutor();   
            }
        });
        */
        /*
         * =================================================================
         * OLD CODE END
         * =================================================================
         */
	}
	
	/*
    * The commented-out property loading and decryption logic remains here.
    * No changes are needed for it.
    */
	/*private static final String TIME_PROPERTIES_FILE_NOT_FOUND = "TimeProperties file not found!!!";
	// ... rest of the old property methods
    */


	/**
	 * This method is responsible for triggering the warning message
	 * */
	
	public void closeAllProcess() {
		
		String[] keySet = (String[]) f1Obj.getWindowMap().keySet().toArray(new String[0]);
		
		for(String  key :keySet) {
			FinanceOne f1= (FinanceOne)f1Obj.getWindowMap().get(key);
			
			if(f1.isParent()) {
				f1.setIsAutoExit(false);
				f1.processCloseRequest(f1.isParent(), f1.isAutoExit());
			}
		}
	}
	
	public void timerEvent() {

		SessionService.sessioOutTimeInitializer();
		f1Obj = new FinanceOne(true);
		
        /*
         * =================================================================
         * OLD CODE START: Both the original 'new Thread()' and the newer
         * 'ExecutorService' approach are replaced by the much simpler
         * Swing Timer below. This removes the need for manual thread
         * management, sleep calls, and invokeLater.
         * =================================================================
         */
		/*
		//old changes start
		//Thread t = new Thread(new Runnable() { ... });
		//t.start();
        // ... JOptionPane logic ...
		//old changes end
		
		//new changes start
		 executor.submit(() -> {
            try {
                Thread.sleep(SessionUtil.msgTime);
            } catch (InterruptedException e2) {
                //logger.info("***** In InterruptedException *****");
                Thread.currentThread().interrupt();
                return;
            }

            SwingUtilities.invokeLater(() -> {
                try {
                    JOptionPane.getRootFrame().dispose();

                    p = JOptionPane.showConfirmDialog(
                            null,
                            SessionUtil.str,
                            SessionConstant.getDialougetitle(),
                            JOptionPane.YES_NO_OPTION
                    );

                    if (p == 0) {
                        closeAllProcess();
                        secureExit(); 
                    } else if (p == 1) {
                        a = 0;
                        f1Obj.menuFlag = false;
                        f1Obj.sessionOut.restart();
                    } else if (p == -1) {
                        closeAllProcess();
                        secureExit();
                    } else {
                        f1Obj.menuFlag = false;
                        f1Obj.initComponents();
                    }
                } catch (Exception ex) {
                    //logger.error("Error in session timeout UI handling", ex);
                }
            });
		});
        //new changes end
        */
        /*
         * =================================================================
         * OLD CODE END
         * =================================================================
         */


        // =================================================================
        // NEW CORRECTED CODE START: Using javax.swing.Timer
        // This is the standard, secure, and simplest way to handle delayed
        // UI events in Swing. It resolves all thread-related security flags.
        // =================================================================
        ActionListener sessionTimeoutListener = new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent evt) {
                // This code runs on the UI thread after the delay automatically.
                p = JOptionPane.showConfirmDialog(
                        null,
                        SessionUtil.str,
                        SessionConstant.getDialougetitle(),
                        JOptionPane.YES_NO_OPTION
                );

                if (p == JOptionPane.YES_OPTION) { // p == 0
                    //logger.debug("AppSession:: User chose YES, closing session.");
                    closeAllProcess();
                    // System.exit() should be avoided. The application will close
                    // when the last window is disposed.
                    // secureExit(); // No longer needed
                } else if (p == JOptionPane.NO_OPTION) { // p == 1
                    //logger.debug("AppSession:: User chose NO, restarting timer.");
                    a = 0;
                    if (f1Obj != null) {
                        f1Obj.menuFlag = false;
                        if (f1Obj.sessionOut != null) {
                           f1Obj.sessionOut.restart();
                        }
                    }
                } else { // User closed the dialog (p == -1) or another case
                    //logger.debug("AppSession:: Dialog closed, closing session.");
                    closeAllProcess();
                    // secureExit(); // No longer needed
                }
            }
        };

        // Create a timer that will fire ONCE after the specified message time.
        Timer sessionTimer = new Timer((int) SessionUtil.msgTime, sessionTimeoutListener);
        sessionTimer.setRepeats(false); // Ensure it only runs once
        sessionTimer.start();
        // =================================================================
        // NEW CORRECTED CODE END
        // =================================================================
	}
	
    /*
     * =================================================================
     * OLD CODE START: This logic for shutting down the executor is no
     * longer necessary because we are using a Swing Timer, which is
     * managed by the framework.
     * =================================================================
     */
	/*
	//new changes start
	public void shutdownExecutor() {
        try {
            executor.shutdown();
            if (!executor.awaitTermination(5, TimeUnit.SECONDS)) {
                executor.shutdownNow();
            }
        } catch (InterruptedException e) {
            executor.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }
	
	
    public void secureExit() {
        try {
            shutdownExecutor(); 
            this.dispose();    
           
        } catch (Exception e) {
//            logger.error("Error during secureExit", e);
        }
    }
    //new changes end
    */
    /*
     * =================================================================
     * OLD CODE END
     * =================================================================
     */
}

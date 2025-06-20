package com.comlinkusa.financeoneui.session;

import java.io.InputStream;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import javax.swing.JFrame;
import javax.swing.JOptionPane;
import javax.swing.SwingUtilities;

import com.comlinkusa.financeoneui.client.FinanceOne;
import com.comlinkusa.financeoneui.logger.ComlinkLogger;

import org.apache.log4j.Logger;

public class AppSession extends JFrame {

    int p;
    public static int a = 0;
    static FinanceOne f1Obj = null;

    static Logger logger = Logger.getLogger(AppSession.class.getName());
    private final ExecutorService executor = Executors.newSingleThreadExecutor();

    static InputStream input = null;

    public AppSession() {
        timerEvent();
    }

    public void timerEvent() {
        try {
            SessionService.sessioOutTimeInitializer();
            f1Obj = new FinanceOne(true);

            executor.submit(() -> {
                try {
                    Thread.sleep(SessionUtil.msgTime);
                } catch (InterruptedException e2) {
                    logger.info("***** In InterruptedException *****");
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
                            secureExit(); // instead of System.exit
                        } else if (p == 1) {
                            a = 0;
                            f1Obj.menuFlag = false;
                            f1Obj.sessionOut.restart();
                        } else if (p == -1) {
                            closeAllProcess();
                            secureExit(); // instead of System.exit
                        } else {
                            f1Obj.menuFlag = false;
                            f1Obj.initComponents();
                        }

                    } catch (Exception ex) {
                        logger.error("Error in session timeout UI handling", ex);
                    }
                });
            });

        } catch (Exception e) {
            logger.error("Error during timer setup: ", e);
        }
    }

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
            shutdownExecutor(); // shut down thread pool
            this.dispose();     // close main window
            // If multiple windows exist, loop through and close them
        } catch (Exception e) {
            logger.error("Error during secureExit", e);
        }
    }

    public void closeAllProcess() {
        String[] keySet = (String[]) f1Obj.getWindowMap().keySet().toArray(new String[0]);

        for (String key : keySet) {
            FinanceOne f1 = (FinanceOne) f1Obj.getWindowMap().get(key);
            if (f1.isParent()) {
                f1.setIsAutoExit(false);
                f1.processCloseRequest(f1.isParent(), f1.isAutoExit());
            }
        }
    }
}
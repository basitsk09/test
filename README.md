import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import javax.swing.JOptionPane;

public class AppSession extends JFrame {

    private static final Logger logger = Logger.getLogger(AppSession.class.getName());
    private final ExecutorService executor = Executors.newSingleThreadExecutor();

    public AppSession() {
        timerEvent();
    }

    public void timerEvent() {
        try {
            SessionService.sessionOutTimeInitializer();

            executor.submit(() -> {
                try {
                    Thread.sleep(SessionService.getSessionTimeOut() * 60 * 1000); // Convert minutes to ms
                } catch (InterruptedException e) {
                    logger.info("Timer thread interrupted: " + e.getMessage());
                    Thread.currentThread().interrupt();
                    return;
                }

                int result = JOptionPane.showConfirmDialog(
                        null,
                        "Session has expired. Do you want to continue?",
                        "Session Timeout",
                        JOptionPane.YES_NO_OPTION
                );

                if (result == JOptionPane.YES_OPTION) {
                    SessionService.sessionOutTimeInitializer();
                } else {
                    SessionService.logoutUser();
                    System.exit(0);
                }
            });

        } catch (Exception e) {
            logger.warning("Error during timer setup: " + e.getMessage());
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
}
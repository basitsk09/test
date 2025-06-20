public void timerEvent() {

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

        // This ensures GUI updates are done on Swing EDT
        SwingUtilities.invokeLater(() -> {
            JOptionPane.getRootFrame().dispose();

            p = JOptionPane.showConfirmDialog(
                    null,
                    SessionUtil.str,
                    SessionConstant.getDialougetitle(),
                    JOptionPane.YES_NO_OPTION
            );

            if (p == 0) {
                closeAllProcess();
            } else if (p == 1) {
                a = 0;
                f1Obj.menuFlag = false;
                f1Obj.sessionOut.restart();
            } else if (p == -1) {
                closeAllProcess();
            } else {
                f1Obj.menuFlag = false;
                f1Obj.initComponents();
            }
        });
    });
}
public AppSession() {
    timerEvent(); // your thread logic

    addWindowListener(new java.awt.event.WindowAdapter() {
        @Override
        public void windowClosing(java.awt.event.WindowEvent e) {
            shutdownExecutor();
            System.exit(0);
        }
    });
}
public static void terminateAppGracefully() {
    for (Window window : Window.getWindows()) {
        window.dispose();
    }
    Runtime.getRuntime().exit(0);
}
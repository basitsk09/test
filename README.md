// --- Helper function to determine the style for the status chip ---
const getStatusStyles = (status) => {
  // Check if the status is 'Pending'
  if (status?.toLowerCase() === 'pending') {
    return {
      backgroundColor: '#ff9800', // Orange
      color: 'white',
      fontWeight: 'bold',
      padding: '4px 12px',
      borderRadius: '16px',
      display: 'inline-block',
      textTransform: 'capitalize',
    };
  }

  // Return a default, minimal style for any other status
  return { padding: '4px 0' };
};

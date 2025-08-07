// #region --- Data Fetching from API ---
useEffect(() => {
  let isMounted = true;
  const fetchData = async () => {
    setIsLoading(true);
    try {
      const payload = {
        qed: user?.quarterEndDate,
        userCapacity: user?.capacity,
      };

      console.log('payload', payload);

      const response = await callApi('/getWriteOffTotalData', payload, 'POST');
      console.log('response', response);
      // ✅ CHANGE 1: Check for the nested "writeOffData" key in the response
      if (isMounted && response?.writeOffData) {
        const dataList = response.writeOffData; // <-- Correct: Access the array inside the object

        // ✅ CHANGE 2: Update mapping to match the new row structure
        const formattedData = dataList.map((row) => ({
          circleCode: row[0].split('~')[0], // <-- Correct: Splits "001~KOLKATA" and takes "001"
          amount: row[1],                   // <-- Correct: Takes the first amount "100"
          status: row[3],                   // <-- Correct: The status is now the 4th item in the array
        }));

        setRows(formattedData);
        setOriginalRows(JSON.parse(JSON.stringify(formattedData)));
      } else if (isMounted) {
        setSnackbarMessage('No data found for the current period.', 'info');
        setRows([]);
        setOriginalRows([]);
      }
    } catch (error) {
      console.error('Failed to fetch write-off data:', error);
      if (isMounted) {
        setSnackbarMessage('Error loading data from the server.', 'error');
      }
    } finally {
      if (isMounted) {
        setIsLoading(false);
      }
    }
  };

  if (user?.quarterEndDate && user?.capacity) {
    fetchData();
  } else {
    setIsLoading(false);
    setSnackbarMessage('User information is missing. Cannot load data.', 'error');
  }

  return () => {
    isMounted = false;
  };
  // ✅ CHANGE 3: Add missing dependencies to prevent stale data issues
}, [user?.quarterEndDate, user?.capacity, callApi, setSnackbarMessage]);
// #endregion

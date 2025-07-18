  const fetchData = async () => {
    let jsonFormData = JSON.stringify({
      quarterEndDate: loggedInUser.quarterEndDate,
    });
    await encrypt(iv, salt, jsonFormData).then(function (result) {
      jsonFormData = result;
    });
    let payload = { iv: ivBase64, salt: saltBase64, data: jsonFormData };
    try {
      const response = await axios.post(
        "/Server/EditBranch/fetchCrsRequests",
        payload,
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        }
      );

      // Transform the raw API data to match the component's expected structure
      const transformedData = response.data.result.map((item) => {
        let changeReqType = "Unknown";
        let branchCode = item.RT_BRANCH;

        // Logic to determine Change Request Type based on RT_TYPE and RT_SUBTYPE
        if (item.RT_TYPE === "A") {
          changeReqType = "Audit Status";
          if (item.RT_SUBTYPE === "M") {
            branchCode = "Multiple Branches";
          }
        } else if (item.RT_TYPE === "B") {
          if (item.RT_SUBTYPE === "A") {
            changeReqType = "Add Branch";
          } else if (item.RT_SUBTYPE === "D") {
            changeReqType = "Delete Branch";
          }
        }

        // Return the object in the format the table expects
        return {
          ID: item.RT_ID,
          CHANGE_REQ_TYPE: changeReqType,
          BRANCH_CODE: branchCode,
          STATUS: String(item.RT_STATUS), // Convert number to string for renderStatusChip
          REQUESTED_USER: item.RT_MAKER,
        };
      });

      setRequestList(transformedData);
      console.log("Transformed data:", transformedData);
      setContent("No pending requests.");
    } catch (error) {
      console.error("Error fetching data:", error);
      setContent("Error fetching data. Please try again.");
    }
  };

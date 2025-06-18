// ... inside submitData function ...

      const dataToSubmit = { ...data };
      for (const key in dataToSubmit) {
        if (Object.prototype.hasOwnProperty.call(dataToSubmit, key)) {
          // Convert all numeric string fields from state to string formatted to 2 decimals for API submission
          if (
            key.endsWith('GrAmt') ||
            key.endsWith('Pro')
          ) {
            // Use parseToDecimal and then toFixed(2) to ensure 2 decimal places as a string
            dataToSubmit[key] = parseToDecimal(dataToSubmit[key]).toFixed(2);
          }
        }
      }

// ... rest of submitData function ...

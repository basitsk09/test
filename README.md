import React, { useState, useEffect, useMemo, useCallback } from 'react';

// Helper component for input cells to keep main component clean
const TableInputCell = ({ name, value, onChange, isReadOnly = false, themeStyles }) => {
  return (
    <td style={themeStyles.tableCell}>
      <input
        type="text"
        name={name}
        value={value}
        onChange={onChange}
        readOnly={isReadOnly}
        className="form-control decimal-2-places" // Keeping class for potential global resets or external libs
        style={isReadOnly ? themeStyles.readOnlyInput : themeStyles.inputField}
        maxLength="18" // Common max length from original JSP
      />
    </td>
  );
};

const Schedule10Form = () => {
  // Initialize state for all form fields.
  // You MUST add ALL fields from your original JSP file here.
  const [formData, setFormData] = useState({
    // Fixed Assets - Tangible Assets
    land1: '',
    building1: '',
    plantNmachinery1: '',
    furniture1: '',
    motorVehicles1: '',
    officeEquip1: '',
    otherPremises1: '',
    electricFitting1: '',
    // Capital Work In Progress
    currYrCwipl1: '',
    currYrCwipl2: '',
    // Investments
    investProp1: '',
    equityShares1: '',
    prefShares1: '',
    govSecurities1: '',
    debentures1: '',
    mutualFunds1: '',
    otherInvest1: '',
    // Inventories
    rawMaterial1: '',
    workInProgress1: '',
    finishedGoods1: '',
    stockInTrade1: '',
    // Trade Receivables
    unsecuredTradeRec1: '',
    securedTradeRec1: '',
    // Cash and Cash Equivalents
    balanceBank1: '',
    chequesDrafts1: '',
    cashHand1: '',
    // Loans
    loanAndAdvance1: '',
    otherCurrentAssets1: '',
    // Totals - these will be calculated
    totalA1: '', // Sum of Tangible Assets
    totalB1: '', // Sum of Investments
    totalC1: '', // Sum of Current Assets
    grandTotal1: '', // Grand Total
    // ... many other fields and calculated totals will go here based on Schedule10.txt
  });

  const [isDarkMode, setIsDarkMode] = useState(() => {
    const savedTheme = localStorage.getItem('theme');
    return savedTheme === 'dark';
  });

  // Define theme styles as an object
  const themeStyles = useMemo(() => ({
    wrapper: {
      minHeight: '100vh',
      display: 'flex',
      flexDirection: 'column',
      backgroundColor: isDarkMode ? '#1a1a1a' : '#f5f5f5',
      color: isDarkMode ? '#e0e0e0' : '#333',
      transition: 'background-color 0.3s ease, color 0.3s ease',
    },
    header: {
      backgroundImage: "url('assets/img/bg2.jpeg')", // Ensure this path is correct
      backgroundSize: 'cover',
      backgroundPosition: 'center',
      padding: '50px 0',
      textAlign: 'center',
      color: 'white',
    },
    headerBrandH3: {
      margin: 0,
      fontSize: '2.5em',
      color: 'white',
    },
    main: {
      backgroundColor: isDarkMode ? '#1a1a1a' : 'white',
      margin: '-20px 0 0',
      borderRadius: '6px',
      boxShadow: '0 16px 24px 2px rgba(0, 0, 0, 0.14), 0 6px 30px 5px rgba(0, 0, 0, 0.12), 0 8px 10px -5px rgba(0, 0, 0, 0.2)',
      zIndex: 1,
      position: 'relative',
      flexGrow: 1,
      transition: 'background-color 0.3s ease',
    },
    section: {
      padding: '30px 0',
    },
    container: {
      maxWidth: '1200px',
      margin: '0 auto',
      padding: '0 15px',
    },
    tableResponsive: {
      width: '100%',
      overflowX: 'auto',
    },
    table: {
      width: '100%',
      borderCollapse: 'collapse',
      marginBottom: '1rem',
      backgroundColor: isDarkMode ? '#1a1a1a' : 'white',
      color: isDarkMode ? '#e0e0e0' : '#333',
    },
    tableTh: {
      padding: '0.75rem',
      verticalAlign: 'top',
      border: `1px solid ${isDarkMode ? '#555' : '#ddd'}`,
      whiteSpace: 'nowrap',
      backgroundColor: isDarkMode ? '#333' : '#e0e0e0',
      textAlign: 'center',
      fontWeight: 'bold',
      color: isDarkMode ? '#e0e0e0' : '#333',
    },
    tableCell: {
      padding: '0.75rem',
      verticalAlign: 'top',
      border: `1px solid ${isDarkMode ? '#555' : '#ddd'}`,
      whiteSpace: 'nowrap',
      textAlign: 'left',
    },
    tableCellRight: {
      padding: '0.75rem',
      verticalAlign: 'top',
      border: `1px solid ${isDarkMode ? '#555' : '#ddd'}`,
      whiteSpace: 'nowrap',
      textAlign: 'right',
    },
    inputField: {
      display: 'block',
      width: '100%',
      padding: '0.375rem 0.75rem',
      fontSize: '1rem',
      lineHeight: '1.5',
      color: isDarkMode ? '#e0e0e0' : '#333',
      backgroundColor: isDarkMode ? '#333' : 'white',
      backgroundClip: 'padding-box',
      border: `1px solid ${isDarkMode ? '#555' : '#ddd'}`,
      borderRadius: '0.25rem',
      textAlign: 'right', // Specific for numerical inputs
      transition: 'border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out, background-color 0.3s ease, color 0.3s ease',
    },
    readOnlyInput: {
      display: 'block',
      width: '100%',
      padding: '0.375rem 0.75rem',
      fontSize: '1rem',
      lineHeight: '1.5',
      color: isDarkMode ? '#e0e0e0' : '#333',
      backgroundColor: isDarkMode ? '#444' : '#e9ecef',
      backgroundClip: 'padding-box',
      border: `1px solid ${isDarkMode ? '#555' : '#ddd'}`,
      borderRadius: '0.25rem',
      opacity: 1,
      textAlign: 'right', // Specific for numerical inputs
      transition: 'background-color 0.3s ease, color 0.3s ease',
    },
    button: {
      display: 'inline-block',
      fontWeight: '400',
      color: 'white',
      textAlign: 'center',
      verticalAlign: 'middle',
      cursor: 'pointer',
      backgroundColor: isDarkMode ? '#0056b3' : '#007bff',
      border: `1px solid ${isDarkMode ? '#0056b3' : '#007bff'}`,
      padding: '0.375rem 0.75rem',
      fontSize: '1rem',
      lineHeight: '1.5',
      borderRadius: '0.25rem',
      transition: 'color 0.15s ease-in-out, background-color 0.15s ease-in-out, border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out',
    },
    darkModeToggle: {
      marginBottom: '1rem',
      textAlign: 'right',
    },
    sectionTitle: {
      display: 'block',
      marginBottom: '5px',
    }
  }), [isDarkMode]); // Re-calculate styles if dark mode changes

  // Apply dark mode class to documentElement (useful if other global styles depend on it)
  // And persist theme preference
  useEffect(() => {
    if (isDarkMode) {
      document.documentElement.classList.add('dark-mode');
      localStorage.setItem('theme', 'dark');
    } else {
      document.documentElement.classList.remove('dark-mode');
      localStorage.setItem('theme', 'light');
    }
  }, [isDarkMode]);

  const toggleDarkMode = () => {
    setIsDarkMode(prevMode => !prevMode);
  };

  // Generic input change handler with basic numeric and 2-decimal validation
  const handleInputChange = useCallback((e) => {
    const { name, value } = e.target;
    // Allow only numbers and up to two decimal places
    const cleanValue = value
      .replace(/[^0-9.]/g, '') // Remove non-numeric except dot
      .replace(/(\..*)\./g, '$1'); // Allow only one dot

    // Further restrict to 2 decimal places if a dot exists
    const parts = cleanValue.split('.');
    let finalValue = cleanValue;
    if (parts.length > 1 && parts[1].length > 2) {
      finalValue = parts[0] + '.' + parts[1].substring(0, 2);
    }

    setFormData(prevData => ({
      ...prevData,
      [name]: finalValue,
    }));
  }, []);

  // --- Calculations using useMemo for performance ---

  // Example: Calculation for totalA1 (Sum of Tangible Assets)
  const calculateTotalA1 = useMemo(() => {
    const fields = [
      formData.land1,
      formData.building1,
      formData.plantNmachinery1,
      formData.furniture1,
      formData.motorVehicles1,
      formData.officeEquip1,
      formData.otherPremises1,
      formData.electricFitting1,
    ];
    const sum = fields.reduce((acc, val) => acc + (parseFloat(val) || 0), 0);
    return sum.toFixed(2);
  }, [
    formData.land1,
    formData.building1,
    formData.plantNmachinery1,
    formData.furniture1,
    formData.motorVehicles1,
    formData.officeEquip1,
    formData.otherPremises1,
    formData.electricFitting1,
  ]);

  // Example: Calculation for totalB1 (Sum of Investments)
  const calculateTotalB1 = useMemo(() => {
    const fields = [
      formData.investProp1,
      formData.equityShares1,
      formData.prefShares1,
      formData.govSecurities1,
      formData.debentures1,
      formData.mutualFunds1,
      formData.otherInvest1,
    ];
    const sum = fields.reduce((acc, val) => acc + (parseFloat(val) || 0), 0);
    return sum.toFixed(2);
  }, [
    formData.investProp1,
    formData.equityShares1,
    formData.prefShares1,
    formData.govSecurities1,
    formData.debentures1,
    formData.mutualFunds1,
    formData.otherInvest1,
  ]);

  // You will need to add MANY more useMemo hooks here for all other calculations:
  // e.g., calculateTotalC1, calculateGrandTotal1, etc., based on your Schedule10.txt

  // --- useEffect to update formData with calculated values ---
  useEffect(() => {
    setFormData(prevData => ({
      ...prevData,
      totalA1: calculateTotalA1,
    }));
  }, [calculateTotalA1]);

  useEffect(() => {
    setFormData(prevData => ({
      ...prevData,
      totalB1: calculateTotalB1,
    }));
  }, [calculateTotalB1]);

  // You will need many more useEffects here to update other calculated fields
  // For complex interdependencies, you might consider a single useEffect that
  // calculates all derived values or use a custom reducer hook if state logic gets very complex.

  // Optional: A submit handler
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log("Form Data Submitted:", formData);
    // Here you would typically send formData to an API
  };

  return (
    <div style={themeStyles.wrapper}>
      <div style={themeStyles.header}>
        <div style={themeStyles.container}>
          <div className="row tim-row">
            <div className="col-md-8 col-md-offset-2">
              <div className="brand">
                <h3 style={themeStyles.headerBrandH3}>Schedule 10</h3>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div style={themeStyles.main}>
        <div style={themeStyles.section}>
          <div style={themeStyles.container}>
            {/* Dark Mode Toggle */}
            <div style={themeStyles.darkModeToggle}>
              <label>
                <input
                  type="checkbox"
                  checked={isDarkMode}
                  onChange={toggleDarkMode}
                />{' '}
                Dark Mode
              </label>
            </div>

            <form onSubmit={handleSubmit}>
              <div style={themeStyles.tableResponsive}>
                <table style={themeStyles.table}>
                  <thead>
                    <tr>
                      <th colSpan="2" style={themeStyles.tableTh}>Particulars</th>
                      <th style={themeStyles.tableTh}>Current Year</th>
                      <th style={themeStyles.tableTh}>Previous Year</th>
                    </tr>
                  </thead>
                  <tbody>
                    {/* Fixed Assets - Tangible Assets */}
                    <tr>
                      <td rowSpan="9" style={{ ...themeStyles.tableCell, textAlign: 'center' }}>
                        <b style={themeStyles.sectionTitle}>Fixed Assets</b><br />
                        Tangible Assets
                      </td>
                      <td style={themeStyles.tableCell}>Land</td>
                      <TableInputCell name="land1" value={formData.land1} onChange={handleInputChange} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td> {/* Placeholder for previous year if needed */}
                    </tr>
                    <tr>
                      <td style={themeStyles.tableCell}>Building</td>
                      <TableInputCell name="building1" value={formData.building1} onChange={handleInputChange} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>
                    <tr>
                      <td style={themeStyles.tableCell}>Plant & Machinery</td>
                      <TableInputCell name="plantNmachinery1" value={formData.plantNmachinery1} onChange={handleInputChange} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>
                    <tr>
                      <td style={themeStyles.tableCell}>Furniture & Fixtures</td>
                      <TableInputCell name="furniture1" value={formData.furniture1} onChange={handleInputChange} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>
                    <tr>
                      <td style={themeStyles.tableCell}>Motor Vehicles</td>
                      <TableInputCell name="motorVehicles1" value={formData.motorVehicles1} onChange={handleInputChange} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>
                    <tr>
                      <td style={themeStyles.tableCell}>Office Equipments</td>
                      <TableInputCell name="officeEquip1" value={formData.officeEquip1} onChange={handleInputChange} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>
                    <tr>
                      <td style={themeStyles.tableCell}>Other Premises</td>
                      <TableInputCell name="otherPremises1" value={formData.otherPremises1} onChange={handleInputChange} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>
                    <tr>
                      <td style={themeStyles.tableCell}>Electric Fitting</td>
                      <TableInputCell name="electricFitting1" value={formData.electricFitting1} onChange={handleInputChange} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>
                    <tr>
                      <td style={themeStyles.tableCell}><b>TOTAL (A)</b></td>
                      <TableInputCell name="totalA1" value={formData.totalA1} isReadOnly={true} onChange={() => {}} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>

                    {/* Capital Work In Progress */}
                    <tr>
                      <td rowSpan="3" style={{ ...themeStyles.tableCell, textAlign: 'center' }}>
                        <b style={themeStyles.sectionTitle}>Capital Work In Progress</b>
                      </td>
                      <td style={themeStyles.tableCell}>Current Year</td>
                      <TableInputCell name="currYrCwipl1" value={formData.currYrCwipl1} onChange={handleInputChange} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>
                    <tr>
                      <td style={themeStyles.tableCell}>Previous Year</td>
                      <TableInputCell name="currYrCwipl2" value={formData.currYrCwipl2} onChange={handleInputChange} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>
                    <tr>
                      <td style={themeStyles.tableCell}><b>TOTAL (B)</b></td>
                      <TableInputCell name="totalB1" value={formData.totalB1} isReadOnly={true} onChange={() => {}} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>
                    {/* ... Continue adding rows for all sections (Investments, Inventories, etc.) */}
                    {/* Remember to replicate the exact HTML structure and field names from your JSP */}

                    {/* Investments */}
                    <tr>
                      <td rowSpan="8" style={{ ...themeStyles.tableCell, textAlign: 'center' }}>
                        <b style={themeStyles.sectionTitle}>Investments</b>
                      </td>
                      <td style={themeStyles.tableCell}>Investment Property</td>
                      <TableInputCell name="investProp1" value={formData.investProp1} onChange={handleInputChange} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>
                    <tr>
                      <td style={themeStyles.tableCell}>Equity Shares</td>
                      <TableInputCell name="equityShares1" value={formData.equityShares1} onChange={handleInputChange} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>
                    <tr>
                      <td style={themeStyles.tableCell}>Preference Shares</td>
                      <TableInputCell name="prefShares1" value={formData.prefShares1} onChange={handleInputChange} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>
                    <tr>
                      <td style={themeStyles.tableCell}>Government Securities</td>
                      <TableInputCell name="govSecurities1" value={formData.govSecurities1} onChange={handleInputChange} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>
                    <tr>
                      <td style={themeStyles.tableCell}>Debentures/Bonds</td>
                      <TableInputCell name="debentures1" value={formData.debentures1} onChange={handleInputChange} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>
                    <tr>
                      <td style={themeStyles.tableCell}>Mutual Funds</td>
                      <TableInputCell name="mutualFunds1" value={formData.mutualFunds1} onChange={handleInputChange} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>
                    <tr>
                      <td style={themeStyles.tableCell}>Other Investments</td>
                      <TableInputCell name="otherInvest1" value={formData.otherInvest1} onChange={handleInputChange} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>
                    <tr>
                      <td style={themeStyles.tableCell}><b>TOTAL (C)</b></td>
                      <TableInputCell name="totalB1" value={formData.totalB1} isReadOnly={true} onChange={() => {}} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>

                    {/* ... Continue with other sections like Inventories, Trade Receivables, etc. */}
                    {/* Ensure all calculations for these sections are also added with useMemo and useEffect */}

                    {/* Grand Total Example */}
                    <tr>
                      <td colSpan="2" style={{ ...themeStyles.tableCellRight, fontWeight: 'bold' }}>GRAND TOTAL</td>
                      <TableInputCell name="grandTotal1" value={formData.grandTotal1} isReadOnly={true} onChange={() => {}} themeStyles={themeStyles} />
                      <td style={themeStyles.tableCell}></td>
                    </tr>
                  </tbody>
                </table>
              </div>
              <button type="submit" style={themeStyles.button}>Submit</button>
            </form>
          </div>
        </div>
      </div>
    </div>
  );
};

export default Schedule10Form;

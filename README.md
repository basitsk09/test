You're absolutely right — in the last complete version of the RW04 React component I provided, the fieldKeyMap and mapped payload preparation were not included yet in the onSave() function. Let's fix that now.


---

✅ Add This to Your Component: fieldKeyMap + Payload Mapping

📍Step 1: Add fieldKeyMap above onSave function

Right after your initialDynamicRow, add:

const fieldKeyMap = {
  particulars: 'particulars',
  provAmtStart: 'provAmt2015',
  writeOff: 'writeOffDur12mon',
  addition: 'additionDur12mon',
  reduction: 'reduInProviAmt',
  provAmtEnd: 'proviAmt2016',
  rate: 'ratePOfProv',
  provRequired: 'provReq',
};


---

📍Step 2: Update onSave() to use it

Replace your existing onSave() function with:

const onSave = () => {
  const mappedDynamicRows = dynamicRows.map(row => {
    const mappedRow = {};
    Object.entries(fieldKeyMap).forEach(([frontendKey, backendKey]) => {
      mappedRow[backendKey] = row[frontendKey] || '';
    });
    return mappedRow;
  });

  console.log('Mapped dynamic data to send to API:', mappedDynamicRows);
  console.log('Static data (raw):', staticData);

  setSnackbar({
    open: true,
    message: 'Data prepared for API. Check console for output.',
    severity: 'success',
  });
};


---

✅ Console Output Example

If your row looks like this:

{
  particulars: 'XYZ',
  provAmtStart: '100',
  writeOff: '10',
  addition: '20',
  reduction: '5',
  provAmtEnd: '105',
  rate: '100',
  provRequired: '105',
}

Then the console output will be:

[
  {
    "particulars": "XYZ",
    "provAmt2015": "100",
    "writeOffDur12mon": "10",
    "additionDur12mon": "20",
    "reduInProviAmt": "5",
    "proviAmt2016": "105",
    "ratePOfProv": "100",
    "provReq": "105"
  }
]

This aligns 100% with your DAO bean expectations.


---

Would you like me to also add static data mapping using a similar approach for backend compatibility (based on IDs like 1_provAmtStart)?




Perfect. Let’s now map the static section data to your DAO’s RW04 bean fields — which expects values like fraudsDebitedProvAfter, fraudsDebitedWrite, etc.


---

✅ Step-by-step Fix for Static Data Mapping


---

🧩 Step 1: Add staticFieldKeyMap

Place this object above onSave(), after fieldKeyMap:

const staticFieldKeyMap = {
  '1': {
    baseName: 'fraudsDebited',
  },
  '1.i': {
    baseName: 'fraudsDebitedPrior100',
  },
  '1.ii': {
    baseName: 'fraudsDebitedDelayed',
  },
  '2': {
    baseName: 'othersRecalled',
  },
  '3': {
    baseName: 'fraudsOthers',
  },
  '3.i': {
    baseName: 'fraudsOthersPrior100',
  },
  '3.ii': {
    baseName: 'fraudsOthersDelayed',
  },
  '4': {
    baseName: 'revenue',
  },
  '5': {
    baseName: 'fslo',
  },
  '6': {
    baseName: 'outstanding',
  },
  '7': {
    baseName: 'npainterest',
  },
};

const staticFieldSuffixMap = {
  provAmtStart: 'ProvAfter',
  writeOff: 'Write',
  addition: 'Addition',
  reduction: 'Reduction',
  provAmtEnd: 'ProvOn',
  rate: 'Rate',
  provRequired: 'ProvReq',
};


---

🧩 Step 2: Add mapStaticToBackend() function

Also above onSave():

const mapStaticToBackend = () => {
  const staticMapped = {};

  Object.keys(staticData).forEach((key) => {
    const [rowId, field] = key.split('_');
    const base = staticFieldKeyMap[rowId]?.baseName;
    const suffix = staticFieldSuffixMap[field];

    if (base && suffix) {
      const backendKey = `${base}${suffix}`;
      staticMapped[backendKey] = staticData[key] || '';
    }
  });

  return staticMapped;
};


---

🧩 Step 3: Final onSave() combining both mappings

Replace your onSave() with this:

const onSave = () => {
  const mappedDynamicRows = dynamicRows.map(row => {
    const mappedRow = {};
    Object.entries(fieldKeyMap).forEach(([frontendKey, backendKey]) => {
      mappedRow[backendKey] = row[frontendKey] || '';
    });
    return mappedRow;
  });

  const mappedStaticRow = mapStaticToBackend();

  console.log('Dynamic Rows Payload (List<RW04>):', mappedDynamicRows);
  console.log('Static Row Payload (RW04):', mappedStaticRow);

  setSnackbar({
    open: true,
    message: 'Mapped data prepared. Check console for API payload.',
    severity: 'success',
  });
};


---

✅ Result

You now have two objects ready:

mappedDynamicRows: for inserting list of rows into CRS_OTH_ASSESTS (id > 18)

mappedStaticRow: for inserting row 1–18 in RW04 Java bean format



---

Would you like me to help implement the fetch or axios POST request that sends this combined payload to your Spring Boot API?


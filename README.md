Yes — based on your code and the fetchReport logic, here’s why "Others" rows under A3 (like in4, out4, out5, etc.) are not showing fetched data:


---

✅ Root Cause

You're only fetching data for rows 1, 2, and 3:

['1', '2', '3'].forEach((num) => {
  fields.forEach((field) => {
    const suffix = backendFieldMap[field];
    nv[`fac${num}`][field] = data[`facility_${suffix}_${num}`] || '';
    nv[`sec${num}`][field] = data[`security_${suffix}_${num}`] || '';
    nv[`in${num}`][field] = data[`sector_${suffix}_a${num}`] || '';
    nv[`out${num}`][field] = data[`sector_${suffix}_b${num}`] || '';
  });
});

> ❌ This skips in4, out4, out5, inTotal, outTotal, etc.




---

✅ Fix: Dynamically Handle All Entry Rows

Replace that loop with one that covers all applicable rowDefinitions dynamically:

🔧 Replace ['1', '2', '3']... block with:

rowDefinitions.forEach(({ id, type }) => {
  if (type !== 'entry') return;

  const fields = ['standard', 'subStandard', 'doubtful', 'loss'];
  if (user.capacity === '61') fields.push('adjustment');

  const base =
    id.startsWith('fac') ? 'facility' :
    id.startsWith('sec') ? 'security' :
    'sector';

  const index = getFieldName(id, 'standard').split('_').pop(); // smart extraction

  fields.forEach((field) => {
    const suffix = backendFieldMap[field];
    const key = `${base}_${suffix}_${index}`;
    nv[id][field] = data[key] || '';
  });
});


---

🧠 Key Notes:

This works for:

fac1 to facTotal

sec1 to secTotal

in1 to in4, inTotal

out1, out2, out3, out4, outTotal, etc.


You reuse your existing getFieldName logic to extract proper field name mapping.

You cover all entry rows dynamically.



---

✅ What You’ll See Working After This:

in4 (“[iv] Others”) under A3a will now show values.

out4, out5, etc. under A3b will also fetch and render data properly.


Let me know if you'd like me to paste the complete updated fetchReport function with this fix applied.


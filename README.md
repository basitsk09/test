const rowDefinitions = [
  { id: 'headerA1', label: 'A-1. Facility Wise Classification', type: 'section' },
  { id: 'fac1', label: '[i] Bills Purchased and Discounted less bills rediscounted $', type: 'entry' },
  {
    id: 'fac2',
    label: '[ii] Cash Credits, Overdrafts, Loans repayable on Demand and Recalled Assets $$',
    type: 'entry',
  },
  { id: 'fac3', label: '[iii]  Term Loans , Agricultural Term Loans, FCNRB Term Loan', type: 'entry' },
  { id: 'facTotal', label: 'Total A-1', type: 'entry' },

  { id: 'headerA2', label: 'A.2 Security Wise Classifications', type: 'section' },
  { id: 'sec1', label: '[i] Secured by Tangible Assets', type: 'entry' },
  { id: 'sec2', label: '[ii] Covered by Bank/DICGC/ECGC/CGTSI/Govt Guarantee', type: 'entry' },
  { id: 'sec3', label: '[iii] Unsecured', type: 'entry' },
  { id: 'secTotal', label: 'Total A-2', type: 'entry' },

  { id: 'headerA3', label: 'A.3 Sector-Wise Classifications', type: 'section' },
  { id: 'headerIn', label: 'a) In India', type: 'subsection' },
  { id: 'in1', label: '[i] Priority', type: 'entry' },
  { id: 'in2', label: '[ii] Public', type: 'entry' },
  { id: 'in3', label: '[iii] Banks in India*', type: 'entry' },
  { id: 'in4', label: '[iv] Others', type: 'entry' },
  { id: 'inTotal', label: 'TOTAL IN INDIA (i+ii+iii+iv)', type: 'entry' },

  { id: 'headerOut', label: 'b) Outside India (Excluding Foreign LCs and BGs)', type: 'subsection' },
  { id: 'out1', label: '[i] Due from Banks', type: 'entry' },
  { id: 'out2', label: '[ii] Due from Others', type: 'entry' },
  { id: 'out3', label: '[1] Bills Purchased and Discounted', type: 'entry' },
  { id: 'out4', label: '[2] Syndicated Loans', type: 'entry' },
  { id: 'out5', label: '[3] Others', type: 'entry' },
  { id: 'outTotal', label: 'TOTAL IN OUTSIDE INDIA (i+ii.1+ii.2+ii.3)', type: 'entry' },

  { id: 'advA3', label: 'Total advances A3 (a+b)', type: 'entry' },
];

// Columns
const columns = [
  { key: 'standard', label: 'Standard', editable: true },
  { key: 'subStandard', label: 'Sub-standard', editable: true },
  { key: 'doubtful', label: 'Doubtful', editable: true },
  { key: 'loss', label: 'Loss', editable: true },
  { key: 'adjustment', label: 'Adjustment', editable: true, conditional: true },
  { key: 'total', label: 'Total', editable: false },
];

const ysaRows = ['fac1', 'fac2', 'fac3', 'facTotal'];

const sectionMap = {
  facTotal: ['fac1', 'fac2', 'fac3'],
  secTotal: ['sec1', 'sec2', 'sec3'],
  inTotal: ['in1', 'in2', 'in3', 'in4'],
  outTotal: ['out1', 'out2', 'out3', 'out4', 'out5'],
  advA3: ['inTotal', 'outTotal'],
};

const getFieldName = (id, key) => {
  let base = id.startsWith('fac') ? 'facility' : id.startsWith('sec') ? 'security' : 'sector';

  const suffix =
    {
      standard: 'Standard',
      subStandard: 'SubStandard',
      doubtful: 'Doubtful',
      loss: 'Loss',
      adjustment: 'Adj',
    }[key] || 'Total';

  let idx;

  if (id === 'inTotal') idx = 'a_Total';
  else if (id === 'outTotal') idx = 'b_Total';
  else if (id === 'advA3') idx = 'ab_Total';
  else if (id.startsWith('in')) idx = `a${id.replace('in', '')}`;
  else if (id.startsWith('out')) idx = `b${id.replace('out', '')}`;
  else if (id.endsWith('Total')) idx = 'Total';
  else idx = id.replace(/\D/g, '');

  return `${base}_${suffix}_${idx}`;
};

const buildSavePayload = (values, circleCode, quarterEndDate, role, save, status) => {
  const payload = { circleCode, quarterEndDate, role, reportName: 'SC9', save, status };
  Object.entries(values).forEach(([id, fields]) => {
    Object.entries(fields).forEach(([key, val]) => {
      // only the numeric inputs, not the YSA column
      if (['standard', 'subStandard', 'doubtful', 'loss', 'adjustment'].includes(key)) {
        // default to '0' if empty
        payload[getFieldName(id, key)] = val || '0';
        console.log(val);
      }
    });
  });
  return payload;
};


///////////////////////////////

error in backend
java.lang.IllegalArgumentException: Unrecognized field "sector_Standard_" (class com.tcs.beans.SC09), not marked as ignorable (125 known properties: "sector_Loss_a_Total", "sector_Standard_ab_Total", "security_Loss_1", "security_Loss_2", "security_Loss_3", "sector_Standard_a_Total", "security_Standard_Total", "security_SubStandard_Total", "sector_SubStandard_a_Total", "security_Adj_Total", "sector_Loss_1", "sector_Loss_2", "sector_Loss_3", "sector_Standard_a1", "facility_Total_1", "facility_Total_2", "facility_Total_3", "sector_Doubtful_a_Total", "facility_Loss_Total", "security_Doubtful_Total", "userId", "facility_Loss_1", "facility_Loss_2", "facility_Loss_3", "sector_SubStandard_1", "sector_SubStandard_2", "sector_Standard_b1", "facility_Standard_Total", "sector_Adj_1", "sector_Adj_2", "sector_Adj_3", "facility_Standard_1", "facility_Standard_2", "facility_Standard_3", "sector_SubStandard_a4", "reportId", "reportMasterId", "role", "sector_Adj_ab_Total", "save", "facility_Doubtful_Total", "security_Loss_Total", "sector_SubStandard_b1", "facility_Doubtful_1", "facility_Doubtful_2", "facility_Doubtful_3", "sector_Loss_a1" [truncated]])
 at [Source: N/A; line: -1, column: -1] (through reference chain: com.tcs.beans.SC09["sector_Standard_"])
	at com.fasterxml.jackson.databind.ObjectMapper._convert(ObjectMapper.java:3589)
	at com.fasterxml.jackson.databind.ObjectMapper.convertValue(ObjectMapper.java:3508)
	at com.tcs.controllers.MakerController.SubmitSC09Report(MakerController.java:1138)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:220)
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:134)
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:116)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:827)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:738)
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:85)
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:963)
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:897)
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:970)
	at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:872)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:665)
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:846)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:750)


 ////
 original paylaod should be this
 {
    "circleCode": "021",
    "quarterEndDate": "31/03/2025",
    "reportId": "125731",
    "userId": "1111111",
    "reportName": "Schedule 9",
    "reportMasterId": "310009",
    "status": "11",
    "areMocPending": true,
    "role": "51",
    "save": true,
    "facility_Adj_1": null,
    "facility_Adj_2": null,
    "facility_Adj_3": null,
    "facility_Adj_Total": "0.00",
    "security_Adj_1": null,
    "security_Adj_2": null,
    "security_Adj_3": null,
    "security_Adj_Total": "0.00",
    "sector_Adj_a1": null,
    "sector_Adj_a2": null,
    "sector_Adj_a3": null,
    "sector_Adj_a4": null,
    "sector_Adj_a_Total": "0.00",
    "sector_Adj_b1": null,
    "sector_Adj_1": null,
    "sector_Adj_2": null,
    "sector_Adj_3": null,
    "sector_Adj_b_Total": "0.00",
    "sector_Adj_ab_Total": "0.00",
    "facility_Standard_1": "0",
    "facility_SubStandard_1": "0",
    "facility_Doubtful_1": "0",
    "facility_Loss_1": "1589085221150.91",
    "facility_Total_1": "1589085221150.91",
    "security_Standard_1": "0",
    "security_SubStandard_1": "0",
    "security_Doubtful_1": "0",
    "security_Loss_1": "5881868903098.4",
    "security_Total_1": "5881868903098.40",
    "sector_Standard_1": "0",
    "sector_SubStandard_1": "0",
    "sector_Doubtful_1": "0",
    "sector_Loss_1": "0",
    "sector_Total_1": "0.00",
    "facility_Standard_2": "0",
    "facility_SubStandard_2": "0",
    "facility_Doubtful_2": "0",
    "facility_Loss_2": "318556021227.78",
    "facility_Total_2": "318556021227.78",
    "security_Standard_2": "0",
    "security_SubStandard_2": "0",
    "security_Doubtful_2": "0",
    "security_Loss_2": "1212121210",
    "security_Total_2": "1212121210.00",
    "facility_Standard_3": "0",
    "facility_SubStandard_3": "0",
    "facility_Doubtful_3": "0",
    "facility_Loss_3": "3974227660719.71",
    "facility_Total_3": "3974227660719.71",
    "security_Standard_3": "0",
    "security_SubStandard_3": "0",
    "security_Doubtful_3": "0",
    "security_Loss_3": "0",
    "security_Total_3": "0.00",
    "facility_Standard_Total": "0.00",
    "facility_SubStandard_Total": "0.00",
    "facility_Doubtful_Total": "0.00",
    "facility_Loss_Total": "5881868903098.40",
    "facility_Total_Total": "5881868903098.40",
    "security_Standard_Total": "0.00",
    "security_SubStandard_Total": "0.00",
    "security_Doubtful_Total": "0.00",
    "security_Loss_Total": "5883081024308.40",
    "security_Total_Total": "5883081024308.40",
    "sector_Standard_a1": "0",
    "sector_SubStandard_a1": "0",
    "sector_Doubtful_a1": "0",
    "sector_Loss_a1": "0",
    "sector_Total_a1": "0.00",
    "sector_Standard_a2": "0",
    "sector_SubStandard_a2": "0",
    "sector_Doubtful_a2": "0",
    "sector_Loss_a2": "0",
    "sector_Total_a2": "0.00",
    "sector_Standard_a3": "0",
    "sector_SubStandard_a3": "0",
    "sector_Doubtful_a3": "0",
    "sector_Loss_a3": "0",
    "sector_Total_a3": "0.00",
    "sector_Standard_a4": "0",
    "sector_SubStandard_a4": "0",
    "sector_Doubtful_a4": "0",
    "sector_Loss_a4": "0",
    "sector_Total_a4": "0.00",
    "sector_Standard_a_Total": "0.00",
    "sector_SubStandard_a_Total": "0.00",
    "sector_Doubtful_a_Total": "0.00",
    "sector_Loss_a_Total": "0.00",
    "sector_Total_a_total": null,
    "sector_Standard_b1": "0",
    "sector_SubStandard_b1": "0",
    "sector_Doubtful_b1": "0",
    "sector_Loss_b1": "5881868903098.4",
    "sector_Total_b1": "5881868903098.40",
    "sector_Standard_b_Total": "0.00",
    "sector_SubStandard_b_Total": "0.00",
    "sector_Doubtful_b_Total": "0.00",
    "sector_Loss_b_Total": "5881868903098.40",
    "sector_Total_b_total": null,
    "sector_Standard_2": "0",
    "sector_SubStandard_2": "0",
    "sector_Doubtful_2": "0",
    "sector_Loss_2": "0",
    "sector_Total_2": "0.00",
    "sector_Standard_3": "0",
    "sector_SubStandard_3": "0",
    "sector_Doubtful_3": "0",
    "sector_Loss_3": "0",
    "sector_Total_3": "0.00",
    "sector_Standard_ab_Total": "0.00",
    "sector_SubStandard_ab_Total": "0.00",
    "sector_Doubtful_ab_Total": "0.00",
    "sector_Loss_ab_Total": "5881868903098.40",
    "sector_Total_ab_Total": null,
    "ccdpFiletimeStamp": null,
    "facility_Total_Display1_SUM": 1589085221150.91,
    "facility_Total_Display2_SUM": 318556021227.78,
    "facility_Total_Display3_SUM": 3974227660719.71,
    "facility_Total_TotalDisplay_SUM": 5881868903098.4,
    "sector_a_Total_a_Total": "0.00",
    "sector_b_Total_b_Total": "5881868903098.40",
    "sector_ab_Total_ab_Total": "5881868903098.40",
    "facility_Total_Display1": "0.00",
    "facility_Total_Display2": "0.00",
    "facility_Total_Display3": "0.00",
    "facility_Total_TotalDisplay": "0.00",
    "FirstTotal": "0",
    "SecondTotal": "0",
    "ThirdTotal": "0"
}


//////////////////

getting this payload


 circleCode = "021"
 quarterEndDate = "31/03/2025"
 reportId = null
 userId = null
 reportName = "SC9"
 reportMasterId = null
 status = "11"
 areMocPending = null
 role = "51"
 save = true
 facility_Adj_1 = "0"
 facility_Adj_2 = "0"
 facility_Adj_3 = "0"
 facility_Adj_Total = "0"
 security_Adj_1 = "0"
 security_Adj_2 = "0"
 security_Adj_3 = "0"
 security_Adj_Total = "0"
 sector_Adj_a1 = "0"
 sector_Adj_a2 = "0"
 sector_Adj_a3 = "0"
 sector_Adj_a4 = "0"
 sector_Adj_a_Total = "0"
 sector_Adj_b1 = "0"
 sector_Adj_1 = "0"
 sector_Adj_2 = "0"
 sector_Adj_3 = "0"
 sector_Adj_b_Total = "0"
 sector_Adj_ab_Total = "0"
 facility_Standard_1 = "0"
 facility_SubStandard_1 = "0"
 facility_Doubtful_1 = "0"
 facility_Loss_1 = "1589085221150.91"
 facility_Total_1 = null
 security_Standard_1 = "0"
 security_SubStandard_1 = "0"
 security_Doubtful_1 = "0"
 security_Loss_1 = "5881868903098.4"
 security_Total_1 = null
 sector_Standard_1 = "0"
 sector_SubStandard_1 = "0"
 sector_Doubtful_1 = "0"
 sector_Loss_1 = "0"
 sector_Total_1 = null
 facility_Standard_2 = "0"
 facility_SubStandard_2 = "0"
 facility_Doubtful_2 = "0"
 facility_Loss_2 = "318556021227.78"
 facility_Total_2 = null
 security_Standard_2 = "0"
 security_SubStandard_2 = "0"
 security_Doubtful_2 = "0"
 security_Loss_2 = "1212121210"
 security_Total_2 = null
 facility_Standard_3 = "0"
 facility_SubStandard_3 = "0"
 facility_Doubtful_3 = "0"
 facility_Loss_3 = "3974227660719.71"
 facility_Total_3 = null
 security_Standard_3 = "0"
 security_SubStandard_3 = "0"
 security_Doubtful_3 = "0"
 security_Loss_3 = "0"
 security_Total_3 = null
 facility_Standard_Total = "0"
 facility_SubStandard_Total = "0"
 facility_Doubtful_Total = "0"
 facility_Loss_Total = "0"
 facility_Total_Total = null
 security_Standard_Total = "0"
 security_SubStandard_Total = "0"
 security_Doubtful_Total = "0"
 security_Loss_Total = "0"
 security_Total_Total = null
 sector_Standard_a1 = "0"
 sector_SubStandard_a1 = "0"
 sector_Doubtful_a1 = "0"
 sector_Loss_a1 = "0"
 sector_Total_a1 = null
 sector_Standard_a2 = "0"
 sector_SubStandard_a2 = "0"
 sector_Doubtful_a2 = "0"
 sector_Loss_a2 = "0"
 sector_Total_a2 = null
 sector_Standard_a3 = "0"
 sector_SubStandard_a3 = "0"
 sector_Doubtful_a3 = "0"
 sector_Loss_a3 = "0"
 sector_Total_a3 = null
 sector_Standard_a4 = "0"
 sector_SubStandard_a4 = "0"
 sector_Doubtful_a4 = "0"
 sector_Loss_a4 = "0"
 sector_Total_a4 = null
 sector_Standard_a_Total = "0"
 sector_SubStandard_a_Total = "0"
 sector_Doubtful_a_Total = "0"
 sector_Loss_a_Total = "0"
 sector_Total_a_total = null
 sector_Standard_b1 = "0"
 sector_SubStandard_b1 = "0"
 sector_Doubtful_b1 = "0"
 sector_Loss_b1 = "5881868903098.4"
 sector_Total_b1 = null
 sector_Standard_b_Total = "0"
 sector_SubStandard_b_Total = "0"
 sector_Doubtful_b_Total = "0"
 sector_Loss_b_Total = "0"
 sector_Total_b_total = null
 sector_Standard_2 = "0"
 sector_SubStandard_2 = "0"
 sector_Doubtful_2 = "0"
 sector_Loss_2 = "0"
 sector_Total_2 = null
 sector_Standard_3 = "0"
 sector_SubStandard_3 = "0"
 sector_Doubtful_3 = "0"
 sector_Loss_3 = "0"
 sector_Total_3 = null
 sector_Standard_ab_Total = "0"
 sector_SubStandard_ab_Total = "0"
 sector_Doubtful_ab_Total = "0"
 sector_Loss_ab_Total = "0"
 sector_Total_ab_Total = null
 ccdpFiletimeStamp = null

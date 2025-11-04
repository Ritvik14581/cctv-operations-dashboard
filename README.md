# Ritvik---Seabird-DSR-Compilation
import React, { useState, useEffect } from 'react';
import { Save, FileText, Calendar, Building2, TrendingUp, Download } from 'lucide-react';

const OperationsDashboard = () => {
  const [viewMode, setViewMode] = useState('edit');
  const [selectedSite, setSelectedSite] = useState('Site 1');
  const [selectedDate, setSelectedDate] = useState(new Date().toISOString().split('T')[0]);
  const [selectedMonth, setSelectedMonth] = useState(new Date().toISOString().slice(0, 7));
  const [sites] = useState(Array.from({ length: 30 }, (_, i) => `Site ${i + 1}`));
  const [dailyData, setDailyData] = useState({});

  const emptyData = {
    keyHighlights: { tbts: { events: 0, attendees: 0 }, eventBrief: { events: 0, attendees: 0 } },
    personnel: {
      authorizedManpower: { onRoll: 0, offRoll: 0 },
      manpowerRequired: { onRoll: 0, offRoll: 0 },
      actualDeployed: { onRoll: 0, offRoll: 0 },
      shortfall: { onRoll: 0, offRoll: 0 },
      totalMovement: { onRoll: 0, offRoll: 0 },
      visitorMovement: 0,
      vipVisitors: 0
    },
    vehicle: { inward: 0, outward: 0 },
    security: { cctv: { available: 0, functional: 0 }, acs: { available: 0, functional: 0 } },
    incidents: { reportableSafety: 0, reportableSecurity: 0, offsite: 0 },
    incidentDetails: ''
  };

  useEffect(() => {
    loadData();
  }, []);

  const loadData = async () => {
    try {
      const result = await window.storage.get('ops-data');
      if (result) setDailyData(JSON.parse(result.value));
    } catch (e) {
      console.log('No data found');
    }
  };

  const saveData = async () => {
    try {
      await window.storage.set('ops-data', JSON.stringify(dailyData));
      alert('Data saved!');
    } catch (e) {
      alert('Save failed');
    }
  };

  const getKey = (date, site) => `${date}_${site}`;
  const getCurrentData = () => dailyData[getKey(selectedDate, selectedSite)] || JSON.parse(JSON.stringify(emptyData));
  
  const updateData = (newData) => {
    setDailyData(prev => ({ ...prev, [getKey(selectedDate, selectedSite)]: newData }));
  };

  const handleChange = (path, value) => {
    const data = getCurrentData();
    const keys = path.split('.');
    let obj = data;
    for (let i = 0; i < keys.length - 1; i++) obj = obj[keys[i]];
    obj[keys[keys.length - 1]] = value;
    updateData(data);
  };

  const aggregateData = (dataArray) => {
    const agg = JSON.parse(JSON.stringify(emptyData));
    const incidents = [];
    
    dataArray.forEach(d => {
      agg.keyHighlights.tbts.events += d.keyHighlights.tbts.events;
      agg.keyHighlights.tbts.attendees += d.keyHighlights.tbts.attendees;
      agg.keyHighlights.eventBrief.events += d.keyHighlights.eventBrief.events;
      agg.keyHighlights.eventBrief.attendees += d.keyHighlights.eventBrief.attendees;
      agg.personnel.authorizedManpower.onRoll += d.personnel.authorizedManpower.onRoll;
      agg.personnel.authorizedManpower.offRoll += d.personnel.authorizedManpower.offRoll;
      agg.personnel.manpowerRequired.onRoll += d.personnel.manpowerRequired.onRoll;
      agg.personnel.manpowerRequired.offRoll += d.personnel.manpowerRequired.offRoll;
      agg.personnel.actualDeployed.onRoll += d.personnel.actualDeployed.onRoll;
      agg.personnel.actualDeployed.offRoll += d.personnel.actualDeployed.offRoll;
      agg.personnel.shortfall.onRoll += d.personnel.shortfall.onRoll;
      agg.personnel.shortfall.offRoll += d.personnel.shortfall.offRoll;
      agg.personnel.totalMovement.onRoll += d.personnel.totalMovement.onRoll;
      agg.personnel.totalMovement.offRoll += d.personnel.totalMovement.offRoll;
      agg.personnel.visitorMovement += d.personnel.visitorMovement;
      agg.personnel.vipVisitors += d.personnel.vipVisitors;
      agg.vehicle.inward += d.vehicle.inward;
      agg.vehicle.outward += d.vehicle.outward;
      agg.security.cctv.available += d.security.cctv.available;
      agg.security.cctv.functional += d.security.cctv.functional;
      agg.security.acs.available += d.security.acs.available;
      agg.security.acs.functional += d.security.acs.functional;
      agg.incidents.reportableSafety += d.incidents.reportableSafety;
      agg.incidents.reportableSecurity += d.incidents.reportableSecurity;
      agg.incidents.offsite += d.incidents.offsite;
      if (d.incidentDetails) incidents.push(d.incidentDetails);
    });
    
    agg.incidentDetails = incidents.join('\n---\n');
    return agg;
  };

  const getDailyCompiled = () => {
    const data = sites.map(s => dailyData[getKey(selectedDate, s)] || emptyData);
    return aggregateData(data);
  };

  const getMonthlyCompiled = () => {
    const [year, month] = selectedMonth.split('-');
    const days = new Date(year, month, 0).getDate();
    const data = [];
    for (let d = 1; d <= days; d++) {
      const date = `${selectedMonth}-${String(d).padStart(2, '0')}`;
      sites.forEach(s => {
        const key = getKey(date, s);
        if (dailyData[key]) data.push(dailyData[key]);
      });
    }
    return aggregateData(data);
  };

  const exportCSV = (data, filename) => {
    const rows = [
      ['Key Highlights - Anchorage & Seabird Marine'],
      ['', 'No of events', 'Attendees'],
      ['TBTs', data.keyHighlights.tbts.events, data.keyHighlights.tbts.attendees],
      ['Event Brief - Mock Drill/Training', data.keyHighlights.eventBrief.events, data.keyHighlights.eventBrief.attendees],
      [],
      ['Personnel Movement', 'On-Roll', 'Off-Roll'],
      ['Authorized Manpower', data.personnel.authorizedManpower.onRoll, data.personnel.authorizedManpower.offRoll],
      ['Manpower Required', data.personnel.manpowerRequired.onRoll, data.personnel.manpowerRequired.offRoll],
      ['Actual Deployed', data.personnel.actualDeployed.onRoll, data.personnel.actualDeployed.offRoll],
      ['Shortfall', data.personnel.shortfall.onRoll, data.personnel.shortfall.offRoll],
      ['Total Movement', data.personnel.totalMovement.onRoll, data.personnel.totalMovement.offRoll],
      ['Visitor Movement', '', data.personnel.visitorMovement],
      ['VIP Visitors', '', data.personnel.vipVisitors],
      [],
      ['Vehicle Movement', 'Inward', 'Outward'],
      ['Dispatch/Receipt', data.vehicle.inward, data.vehicle.outward],
      [],
      ['Security Automation', 'Available', 'Functional'],
      ['CCTV', data.security.cctv.available, data.security.cctv.functional],
      ['ACS', data.security.acs.available, data.security.acs.functional],
      [],
      ['Incident Summary'],
      ['Safety', data.incidents.reportableSafety],
      ['Security', data.incidents.reportableSecurity],
      ['Offsite', data.incidents.offsite],
      [],
      ['Incident Details', data.incidentDetails]
    ];
    const csv = rows.map(r => r.join(',')).join('\n');
    const blob = new Blob([csv], { type: 'text/csv' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = filename;
    a.click();
  };

  const ReportTable = ({ data, title }) => (
    <div className="bg-white rounded-lg shadow-lg overflow-hidden mb-6">
      <div className="bg-blue-900 text-white p-4">
        <h2 className="text-xl font-bold">{title}</h2>
      </div>
      <table className="w-full border-collapse text-sm">
        <thead>
          <tr className="bg-blue-800 text-white">
            <th colSpan="3" className="p-2 text-left border border-gray-300">Key Highlights</th>
          </tr>
        </thead>
        <tbody>
          <tr className="bg-gray-100">
            <td className="p-2 border font-semibold"></td>
            <td className="p-2 border font-semibold text-center">Events</td>
            <td className="p-2 border font-semibold text-center">Attendees</td>
          </tr>
          <tr>
            <td className="p-2 border">TBTs</td>
            <td className="p-2 border text-center">{data.keyHighlights.tbts.events}</td>
            <td className="p-2 border text-center">{data.keyHighlights.tbts.attendees}</td>
          </tr>
          <tr>
            <td className="p-2 border">Event Brief</td>
            <td className="p-2 border text-center">{data.keyHighlights.eventBrief.events}</td>
            <td className="p-2 border text-center">{data.keyHighlights.eventBrief.attendees}</td>
          </tr>
          <tr className="bg-blue-700 text-white">
            <td colSpan="3" className="p-2 border font-bold text-center">Personnel Movement</td>
          </tr>
          <tr className="bg-gray-100">
            <td className="p-2 border"></td>
            <td className="p-2 border font-semibold text-center">On-Roll</td>
            <td className="p-2 border font-semibold text-center">Off-Roll</td>
          </tr>
          <tr>
            <td className="p-2 border">Authorized Manpower</td>
            <td className="p-2 border text-center">{data.personnel.authorizedManpower.onRoll}</td>
            <td className="p-2 border text-center">{data.personnel.authorizedManpower.offRoll}</td>
          </tr>
          <tr>
            <td className="p-2 border">Manpower Required</td>
            <td className="p-2 border text-center">{data.personnel.manpowerRequired.onRoll}</td>
            <td className="p-2 border text-center">{data.personnel.manpowerRequired.offRoll}</td>
          </tr>
          <tr>
            <td className="p-2 border">Actual Deployed</td>
            <td className="p-2 border text-center">{data.personnel.actualDeployed.onRoll}</td>
            <td className="p-2 border text-center">{data.personnel.actualDeployed.offRoll}</td>
          </tr>
          <tr>
            <td className="p-2 border">Shortfall</td>
            <td className="p-2 border text-center">{data.personnel.shortfall.onRoll}</td>
            <td className="p-2 border text-center">{data.personnel.shortfall.offRoll}</td>
          </tr>
          <tr className="bg-gray-50">
            <td className="p-2 border font-semibold">Total Movement</td>
            <td className="p-2 border text-center font-semibold">{data.personnel.totalMovement.onRoll}</td>
            <td className="p-2 border text-center font-semibold">{data.personnel.totalMovement.offRoll}</td>
          </tr>
          <tr>
            <td className="p-2 border">Visitor Movement</td>
            <td colSpan="2" className="p-2 border text-center">{data.personnel.visitorMovement}</td>
          </tr>
          <tr>
            <td className="p-2 border">VIP Visitors</td>
            <td colSpan="2" className="p-2 border text-center">{data.personnel.vipVisitors}</td>
          </tr>
          <tr className="bg-blue-700 text-white">
            <td colSpan="3" className="p-2 border font-bold text-center">Vehicle Movement</td>
          </tr>
          <tr className="bg-gray-100">
            <td className="p-2 border"></td>
            <td className="p-2 border font-semibold text-center">Inward</td>
            <td className="p-2 border font-semibold text-center">Outward</td>
          </tr>
          <tr>
            <td className="p-2 border">Dispatch/Receipt</td>
            <td className="p-2 border text-center">{data.vehicle.inward}</td>
            <td className="p-2 border text-center">{data.vehicle.outward}</td>
          </tr>
          <tr className="bg-blue-700 text-white">
            <td colSpan="3" className="p-2 border font-bold text-center">Security Automation</td>
          </tr>
          <tr className="bg-gray-100">
            <td className="p-2 border"></td>
            <td className="p-2 border font-semibold text-center">Available</td>
            <td className="p-2 border font-semibold text-center">Functional</td>
          </tr>
          <tr>
            <td className="p-2 border">CCTV</td>
            <td className="p-2 border text-center">{data.security.cctv.available}</td>
            <td className="p-2 border text-center">{data.security.cctv.functional}</td>
          </tr>
          <tr>
            <td className="p-2 border">ACS</td>
            <td className="p-2 border text-center">{data.security.acs.available}</td>
            <td className="p-2 border text-center">{data.security.acs.functional}</td>
          </tr>
          <tr className="bg-blue-700 text-white">
            <td colSpan="3" className="p-2 border font-bold text-center">Incidents</td>
          </tr>
          <tr>
            <td className="p-2 border">Safety</td>
            <td colSpan="2" className="p-2 border text-center">{data.incidents.reportableSafety}</td>
          </tr>
          <tr>
            <td className="p-2 border">Security</td>
            <td colSpan="2" className="p-2 border text-center">{data.incidents.reportableSecurity}</td>
          </tr>
          <tr>
            <td className="p-2 border">Offsite</td>
            <td colSpan="2" className="p-2 border text-center">{data.incidents.offsite}</td>
          </tr>
          <tr className="bg-blue-700 text-white">
            <td colSpan="3" className="p-2 border font-bold text-center">Incident Details</td>
          </tr>
          <tr>
            <td colSpan="3" className="p-2 border whitespace-pre-wrap">{data.incidentDetails || 'None'}</td>
          </tr>
        </tbody>
      </table>
    </div>
  );

  const InputField = ({ label, value, onChange, type = "number" }) => (
    <div>
      <label className="block text-xs font-semibold mb-1">{label}</label>
      <input type={type} value={value} onChange={onChange} className="w-full p-2 border rounded text-sm" />
    </div>
  );

  const data = getCurrentData();

  return (
    <div className="min-h-screen bg-gray-50 p-4">
      <div className="max-w-7xl mx-auto">
        <div className="bg-white rounded-lg shadow p-4 mb-4">
          <h1 className="text-2xl font-bold text-blue-900 mb-1">Operations Dashboard</h1>
          <p className="text-gray-600 text-sm">Anchorage & Seabird Marine</p>
        </div>

        <div className="flex flex-wrap gap-2 mb-4">
          <button onClick={() => setViewMode('edit')} className={`px-4 py-2 rounded font-semibold ${viewMode === 'edit' ? 'bg-blue-700 text-white' : 'bg-white'}`}>
            Edit Data
          </button>
          <button onClick={() => setViewMode('daily')} className={`px-4 py-2 rounded font-semibold ${viewMode === 'daily' ? 'bg-blue-700 text-white' : 'bg-white'}`}>
            Daily Report
          </button>
          <button onClick={() => setViewMode('monthly')} className={`px-4 py-2 rounded font-semibold ${viewMode === 'monthly' ? 'bg-blue-700 text-white' : 'bg-white'}`}>
            Monthly Report
          </button>
          <button onClick={saveData} className="px-4 py-2 rounded font-semibold bg-green-600 text-white ml-auto">
            <Save size={16} className="inline mr-1" /> Save
          </button>
        </div>

        {viewMode === 'edit' && (
          <div className="space-y-4">
            <div className="bg-white rounded-lg shadow p-4">
              <div className="grid grid-cols-2 gap-4">
                <InputField label="Date" type="date" value={selectedDate} onChange={e => setSelectedDate(e.target.value)} />
                <div>
                  <label className="block text-xs font-semibold mb-1">Site</label>
                  <select value={selectedSite} onChange={e => setSelectedSite(e.target.value)} className="w-full p-2 border rounded text-sm">
                    {sites.map(s => <option key={s} value={s}>{s}</option>)}
                  </select>
                </div>
              </div>
            </div>

            <div className="bg-white rounded-lg shadow p-4">
              <h3 className="font-bold text-blue-800 mb-3">Key Highlights</h3>
              <div className="grid grid-cols-2 gap-3">
                <InputField label="TBTs - Events" value={data.keyHighlights.tbts.events} onChange={e => handleChange('keyHighlights.tbts.events', +e.target.value || 0)} />
                <InputField label="TBTs - Attendees" value={data.keyHighlights.tbts.attendees} onChange={e => handleChange('keyHighlights.tbts.attendees', +e.target.value || 0)} />
                <InputField label="Event Brief - Events" value={data.keyHighlights.eventBrief.events} onChange={e => handleChange('keyHighlights.eventBrief.events', +e.target.value || 0)} />
                <InputField label="Event Brief - Attendees" value={data.keyHighlights.eventBrief.attendees} onChange={e => handleChange('keyHighlights.eventBrief.attendees', +e.target.value || 0)} />
              </div>
            </div>

            <div className="bg-white rounded-lg shadow p-4">
              <h3 className="font-bold text-blue-800 mb-3">Personnel</h3>
              <div className="grid grid-cols-2 gap-3">
                <InputField label="Auth Manpower - On Roll" value={data.personnel.authorizedManpower.onRoll} onChange={e => handleChange('personnel.authorizedManpower.onRoll', +e.target.value || 0)} />
                <InputField label="Auth Manpower - Off Roll" value={data.personnel.authorizedManpower.offRoll} onChange={e => handleChange('personnel.authorizedManpower.offRoll', +e.target.value || 0)} />
                <InputField label="Required - On Roll" value={data.personnel.manpowerRequired.onRoll} onChange={e => handleChange('personnel.manpowerRequired.onRoll', +e.target.value || 0)} />
                <InputField label="Required - Off Roll" value={data.personnel.manpowerRequired.offRoll} onChange={e => handleChange('personnel.manpowerRequired.offRoll', +e.target.value || 0)} />
                <InputField label="Deployed - On Roll" value={data.personnel.actualDeployed.onRoll} onChange={e => handleChange('personnel.actualDeployed.onRoll', +e.target.value || 0)} />
                <InputField label="Deployed - Off Roll" value={data.personnel.actualDeployed.offRoll} onChange={e => handleChange('personnel.actualDeployed.offRoll', +e.target.value || 0)} />
                <InputField label="Shortfall - On Roll" value={data.personnel.shortfall.onRoll} onChange={e => handleChange('personnel.shortfall.onRoll', +e.target.value || 0)} />
                <InputField label="Shortfall - Off Roll" value={data.personnel.shortfall.offRoll} onChange={e => handleChange('personnel.shortfall.offRoll', +e.target.value || 0)} />
                <InputField label="Total Movement - On Roll" value={data.personnel.totalMovement.onRoll} onChange={e => handleChange('personnel.totalMovement.onRoll', +e.target.value || 0)} />
                <InputField label="Total Movement - Off Roll" value={data.personnel.totalMovement.offRoll} onChange={e => handleChange('personnel.totalMovement.offRoll', +e.target.value || 0)} />
                <InputField label="Visitor Movement" value={data.personnel.visitorMovement} onChange={e => handleChange('personnel.visitorMovement', +e.target.value || 0)} />
                <InputField label="VIP Visitors" value={data.personnel.vipVisitors} onChange={e => handleChange('personnel.vipVisitors', +e.target.value || 0)} />
              </div>
            </div>

            <div className="bg-white rounded-lg shadow p-4">
              <h3 className="font-bold text-blue-800 mb-3">Vehicle & Security</h3>
              <div className="grid grid-cols-2 gap-3">
                <InputField label="Vehicle Inward" value={data.vehicle.inward} onChange={e => handleChange('vehicle.inward', +e.target.value || 0)} />
                <InputField label="Vehicle Outward" value={data.vehicle.outward} onChange={e => handleChange('vehicle.outward', +e.target.value || 0)} />
                <InputField label="CCTV Available" value={data.security.cctv.available} onChange={e => handleChange('security.cctv.available', +e.target.value || 0)} />
                <InputField label="CCTV Functional" value={data.security.cctv.functional} onChange={e => handleChange('security.cctv.functional', +e.target.value || 0)} />
                <InputField label="ACS Available" value={data.security.acs.available} onChange={e => handleChange('security.acs.available', +e.target.value || 0)} />
                <InputField label="ACS Functional" value={data.security.acs.functional} onChange={e => handleChange('security.acs.functional', +e.target.value || 0)} />
              </div>
            </div>

            <div className="bg-white rounded-lg shadow p-4">
              <h3 className="font-bold text-blue-800 mb-3">Incidents</h3>
              <div className="grid grid-cols-3 gap-3 mb-3">
                <InputField label="Safety" value={data.incidents.reportableSafety} onChange={e => handleChange('incidents.reportableSafety', +e.target.value || 0)} />
                <InputField label="Security" value={data.incidents.reportableSecurity} onChange={e => handleChange('incidents.reportableSecurity', +e.target.value || 0)} />
                <InputField label="Offsite" value={data.incidents.offsite} onChange={e => handleChange('incidents.offsite', +e.target.value || 0)} />
              </div>
              <label className="block text-xs font-semibold mb-1">Details</label>
              <textarea value={data.incidentDetails} onChange={e => handleChange('incidentDetails', e.target.value)} className="w-full p-2 border rounded text-sm h-24" />
            </div>
          </div>
        )}

        {viewMode === 'daily' && (
          <div>
            <div className="bg-white rounded-lg shadow p-4 mb-4">
              <div className="flex gap-4 items-end">
                <InputField label="Select Date" type="date" value={selectedDate} onChange={e => setSelectedDate(e.target.value)} />
                <button onClick={() => exportCSV(getDailyCompiled(), `daily_${selectedDate}.csv`)} className="px-4 py-2 bg-green-600 text-white rounded">
                  <Download size={16} className="inline mr-1" /> Export
                </button>
              </div>
            </div>
            <ReportTable data={getDailyCompiled()} title={`Daily Compiled Report - ${selectedDate}`} />
          </div>
        )}

        {viewMode === 'monthly' && (
          <div>
            <div className="bg-white rounded-lg shadow p-4 mb-4">
              <div className="flex gap-4 items-end">
                <InputField label="Select Month" type="month" value={selectedMonth} onChange={e => setSelectedMonth(e.target.value)} />
                <button onClick={() => exportCSV(getMonthlyCompiled(), `monthly_${selectedMonth}.csv`)} className="px-4 py-2 bg-green-600 text-white rounded">
                  <Download size={16} className="inline mr-1" /> Export
                </button>
              </div>
            </div>
            <ReportTable data={getMonthlyCompiled()} title={`Monthly Compiled Report - ${selectedMonth}`} />
          </div>
        )}
      </div>
    </div>
  );
};

export default OperationsDashboard;

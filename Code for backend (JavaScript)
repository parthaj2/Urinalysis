import React, { useState } from 'react';
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card';
import { Alert, AlertDescription } from '@/components/ui/alert';
import { Check, AlertTriangle, Lock, Users, RefreshCw } from 'lucide-react';

const ArduinoUrinalysisSystem = () => {
  // User authentication state
  const [currentUser, setCurrentUser] = useState({
    id: 'doc123',
    role: 'provider',
    name: 'Dr. Smith'
  });

  // Patient records state
  const [patients, setPatients] = useState([
    {
      id: 'pat001',
      name: 'John Doe',
      records: []
    },
    {
      id: 'pat002',
      name: 'Jane Smith',
      records: []
    }
  ]);

  // Selected patient state
  const [selectedPatient, setSelectedPatient] = useState(null);

  // New record state
  const [newRecord, setNewRecord] = useState({
    date: '',
    hemoglobin: '',
    bilirubin: '',
    porphyria: '',
    notes: ''
  });

  // Arduino reading states
  const [arduinoReading, setArduinoReading] = useState(null);
  const [isReading, setIsReading] = useState(false);
  const [readingError, setReadingError] = useState(null);

  // Reference ranges
  const normalRanges = {
    hemoglobin: '0-0.3 mg/dL',
    bilirubin: '0.2-1.2 mg/dL',
    porphyria: '<20 μg/24h'
  };

  // Handle record submission
  const handleSubmitRecord = () => {
    if (!selectedPatient) return;

    setPatients(prevPatients => {
      return prevPatients.map(patient => {
        if (patient.id === selectedPatient.id) {
          return {
            ...patient,
            records: [...patient.records, {
              ...newRecord,
              id: Date.now(),
              providerId: currentUser.id,
              providerName: currentUser.name,
              timestamp: new Date().toISOString()
            }]
          };
        }
        return patient;
      });
    });

    // Reset form
    setNewRecord({
      date: '',
      hemoglobin: '',
      bilirubin: '',
      porphyria: '',
      notes: ''
    });
  };

  // Fetch reading from Arduino
  const fetchArduinoReading = async () => {
    try {
      setIsReading(true);
      setReadingError(null);
      const response = await fetch('http://localhost:3001/api/reading');
      if (!response.ok) throw new Error('Failed to fetch reading');
      const data = await response.json();
      setArduinoReading(data);
      
      // Auto-fill the form with Arduino readings
      setNewRecord(prev => ({
        ...prev,
        hemoglobin: data.hemoglobin.toFixed(3),
        bilirubin: data.bilirubin.toFixed(3),
        porphyria: data.porphyria.toFixed(3),
      }));
    } catch (error) {
      setReadingError(error.message);
    } finally {
      setIsReading(false);
    }
  };

  // Determine if value is within normal range
  const isInRange = (parameter, value) => {
    if (!value) return true;
    switch (parameter) {
      case 'hemoglobin':
        return parseFloat(value) <= 0.3;
      case 'bilirubin':
        return parseFloat(value) >= 0.2 && parseFloat(value) <= 1.2;
      case 'porphyria':
        return parseFloat(value) < 20;
      default:
        return true;
    }
  };

  return (
    <div className="w-full max-w-6xl mx-auto p-4">
      <Card className="mb-4">
        <CardHeader className="flex flex-row items-center justify-between">
          <CardTitle className="text-2xl font-bold">Urinalysis Management System</CardTitle>
          <div className="flex items-center space-x-2">
            <Lock className="w-5 h-5" />
            <span className="text-sm">{currentUser.name}</span>
          </div>
        </CardHeader>
      </Card>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        {/* Patient Selection */}
        <Card className="md:col-span-1">
          <CardHeader>
            <CardTitle className="flex items-center">
              <Users className="w-5 h-5 mr-2" />
              Patients
            </CardTitle>
          </CardHeader>
          <CardContent>
            <div className="space-y-2">
              {patients.map(patient => (
                <button
                  key={patient.id}
                  onClick={() => setSelectedPatient(patient)}
                  className={`w-full p-2 text-left rounded-md ${
                    selectedPatient?.id === patient.id
                      ? 'bg-blue-100'
                      : 'hover:bg-gray-100'
                  }`}
                >
                  {patient.name}
                </button>
              ))}
            </div>
          </CardContent>
        </Card>

        {/* Record Entry */}
        <Card className="md:col-span-2">
          <CardHeader>
            <CardTitle>
              {selectedPatient 
                ? `New Record for ${selectedPatient.name}`
                : 'Select a patient'}
            </CardTitle>
          </CardHeader>
          <CardContent>
            {selectedPatient ? (
              <div className="space-y-4">
                <button
                  onClick={fetchArduinoReading}
                  disabled={isReading}
                  className="flex items-center justify-center w-full p-2 mb-4 bg-green-500 text-white rounded-md hover:bg-green-600"
                >
                  <RefreshCw className={`w-4 h-4 mr-2 ${isReading ? 'animate-spin' : ''}`} />
                  {isReading ? 'Reading...' : 'Get Arduino Reading'}
                </button>

                {readingError && (
                  <Alert className="mb-4">
                    <AlertDescription className="text-red-500">
                      Error reading from Arduino: {readingError}
                    </AlertDescription>
                  </Alert>
                )}

                <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                  <div>
                    <label className="block text-sm font-medium mb-1">Date</label>
                    <input
                      type="date"
                      className="w-full p-2 border rounded-md"
                      value={newRecord.date}
                      onChange={e => setNewRecord({...newRecord, date: e.target.value})}
                    />
                  </div>

                  {['hemoglobin', 'bilirubin', 'porphyria'].map(param => (
                    <div key={param}>
                      <label className="block text-sm font-medium mb-1">
                        {param.charAt(0).toUpperCase() + param.slice(1)}
                      </label>
                      <div className="relative">
                        <input
                          type="number"
                          step="0.1"
                          className="w-full p-2 border rounded-md"
                          value={newRecord[param]}
                          onChange={e => setNewRecord({...newRecord, [param]: e.target.value})}
                        />
                        {newRecord[param] && (
                          <div className="flex items-center mt-1">
                            {isInRange(param, newRecord[param]) ? (
                              <Check className="w-4 h-4 text-green-500" />
                            ) : (
                              <AlertTriangle className="w-4 h-4 text-yellow-500" />
                            )}
                            <span className="text-xs ml-1">
                              Normal: {normalRanges[param]}
                            </span>
                          </div>
                        )}
                      </div>
                    </div>
                  ))}

                  <div className="md:col-span-2">
                    <label className="block text-sm font-medium mb-1">Notes</label>
                    <textarea
                      className="w-full p-2 border rounded-md"
                      rows="3"
                      value={newRecord.notes}
                      onChange={e => setNewRecord({...newRecord, notes: e.target.value})}
                    />
                  </div>
                </div>

                <button
                  onClick={handleSubmitRecord}
                  className="w-full bg-blue-500 text-white p-2 rounded-md hover:bg-blue-600"
                >
                  Save Record
                </button>
              </div>
            ) : (
              <Alert>
                <AlertDescription>
                  Please select a patient from the list to add a new record.
                </AlertDescription>
              </Alert>
            )}
          </CardContent>
        </Card>

        {/* Patient History */}
        {selectedPatient && (
          <Card className="md:col-span-3">
            <CardHeader>
              <CardTitle>Patient History - {selectedPatient.name}</CardTitle>
            </CardHeader>
            <CardContent>
              <div className="overflow-x-auto">
                <table className="w-full">
                  <thead>
                    <tr>
                      <th className="text-left p-2">Date</th>
                      <th className="text-left p-2">Hemoglobin</th>
                      <th className="text-left p-2">Bilirubin</th>
                      <th className="text-left p-2">Porphyria</th>
                      <th className="text-left p-2">Provider</th>
                      <th className="text-left p-2">Notes</th>
                    </tr>
                  </thead>
                  <tbody>
                    {selectedPatient.records.map(record => (
                      <tr key={record.id} className="border-t">
                        <td className="p-2">{record.date}</td>
                        <td className="p-2">{record.hemoglobin} mg/dL</td>
                        <td className="p-2">{record.bilirubin} mg/dL</td>
                        <td className="p-2">{record.porphyria} μg/24h</td>
                        <td className="p-2">{record.providerName}</td>
                        <td className="p-2">{record.notes}</td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            </CardContent>
          </Card>
        )}
      </div>
    </div>
  );
};

export default ArduinoUrinalysisSystem;

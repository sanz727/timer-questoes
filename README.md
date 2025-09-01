import React, { useState, useEffect, useRef } from 'react';
import { Play, Pause, Square, BarChart3, Download, Clock, Target, TrendingUp, CheckCircle, XCircle } from 'lucide-react';

const QuestionTimer = () => {
  const [time, setTime] = useState(0);
  const [isRunning, setIsRunning] = useState(false);
  const [sessions, setSessions] = useState([]);
  const [currentQuestion, setCurrentQuestion] = useState('');
  const [showStats, setShowStats] = useState(false);
  const intervalRef = useRef(null);

  useEffect(() => {
    if (isRunning) {
      intervalRef.current = setInterval(() => {
        setTime(prevTime => prevTime + 1);
      }, 1000);
    } else {
      clearInterval(intervalRef.current);
    }

    return () => clearInterval(intervalRef.current);
  }, [isRunning]);

  const formatTime = (seconds) => {
    const mins = Math.floor(seconds / 60);
    const secs = seconds % 60;
    return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
  };

  const startTimer = () => {
    setIsRunning(true);
  };

  const pauseTimer = () => {
    setIsRunning(false);
  };

  const stopTimer = () => {
    setIsRunning(false);
    if (time > 0) {
      const newSession = {
        id: Date.now(),
        question: currentQuestion || `Questão ${sessions.length + 1}`,
        timeInSeconds: time,
        formattedTime: formatTime(time),
        timestamp: new Date().toLocaleString('pt-BR'),
        correct: null
      };
      setSessions(prev => [...prev, newSession]);
      setTime(0);
      setCurrentQuestion('');
    }
  };

  const markAnswer = (sessionId, isCorrect) => {
    setSessions(prev => prev.map(session => 
      session.id === sessionId 
        ? { ...session, correct: isCorrect }
        : session
    ));
  };

  const deleteSession = (sessionId) => {
    setSessions(prev => prev.filter(session => session.id !== sessionId));
  };

  const calculateStats = () => {
    if (sessions.length === 0) return null;

    const totalTime = sessions.reduce((sum, session) => sum + session.timeInSeconds, 0);
    const avgTime = totalTime / sessions.length;
    const answeredQuestions = sessions.filter(s => s.correct !== null);
    const correctAnswers = sessions.filter(s => s.correct === true).length;
    const accuracy = answeredQuestions.length > 0 ? (correctAnswers / answeredQuestions.length) * 100 : 0;

    return {
      totalQuestions: sessions.length,
      totalTime: formatTime(totalTime),
      avgTime: formatTime(Math.round(avgTime)),
      accuracy: accuracy.toFixed(1),
      correctAnswers,
      totalAnswered: answeredQuestions.length
    };
  };

  const exportToNotion = () => {
    const stats = calculateStats();
    const data = {
      sessions: sessions.map(s => ({
        Questão: s.question,
        'Tempo (min:seg)': s.formattedTime,
        'Tempo (segundos)': s.timeInSeconds,
        Data: s.timestamp,
        Acertou: s.correct === true ? 'Sim' : s.correct === false ? 'Não' : 'N/A'
      })),
      estatisticas: stats
    };

    const jsonData = JSON.stringify(data, null, 2);
    const blob = new Blob([jsonData], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `timer-questoes-${new Date().toISOString().split('T')[0]}.json`;
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
  };

  const stats = calculateStats();

  return (
    <div className="max-w-4xl mx-auto p-6 bg-gradient-to-br from-blue-50 to-indigo-100 min-h-screen">
      <div className="bg-white rounded-2xl shadow-xl p-8">
        <h1 className="text-3xl font-bold text-center mb-8 text-gray-800 flex items-center justify-center gap-3">
          <Clock className="text-blue-600" />
          Timer de Questões
        </h1>

        {/* Timer Principal */}
        <div className="text-center mb-8">
          <div className="text-6xl font-mono font-bold text-gray-800 mb-4">
            {formatTime(time)}
          </div>
          
          <input
            type="text"
            placeholder="Nome da questão (opcional)"
            value={currentQuestion}
            onChange={(e) => setCurrentQuestion(e.target.value)}
            className="w-full max-w-md mx-auto px-4 py-2 border border-gray-300 rounded-lg mb-4 text-center"
          />

          <div className="flex justify-center gap-4">
            <button
              onClick={isRunning ? pauseTimer : startTimer}
              className={`flex items-center gap-2 px-6 py-3 rounded-lg font-semibold transition-colors ${
                isRunning 
                  ? 'bg-orange-500 hover:bg-orange-600 text-white' 
                  : 'bg-green-500 hover:bg-green-600 text-white'
              }`}
            >
              {isRunning ? <Pause size={20} /> : <Play size={20} />}
              {isRunning ? 'Pausar' : 'Iniciar'}
            </button>
            
            <button
              onClick={stopTimer}
              className="flex items-center gap-2 px-6 py-3 bg-red-500 hover:bg-red-600 text-white rounded-lg font-semibold transition-colors"
              disabled={time === 0}
            >
              <Square size={20} />
              Finalizar
            </button>
          </div>
        </div>

        {/* Estatísticas */}
        {stats && (
          <div className="bg-gray-50 rounded-xl p-6 mb-8">
            <div className="flex items-center justify-between mb-4">
              <h2 className="text-xl font-bold text-gray-800 flex items-center gap-2">
                <BarChart3 className="text-blue-600" />
                Estatísticas
              </h2>
              <div className="flex gap-2">
                <button
                  onClick={() => setShowStats(!showStats)}
                  className="px-4 py-2 bg-blue-500 hover:bg-blue-600 text-white rounded-lg font-semibold transition-colors"
                >
                  {showStats ? 'Ocultar' : 'Mostrar'} Detalhes
                </button>
                <button
                  onClick={exportToNotion}
                  className="flex items-center gap-2 px-4 py-2 bg-purple-500 hover:bg-purple-600 text-white rounded-lg font-semibold transition-colors"
                >
                  <Download size={16} />
                  Exportar
                </button>
              </div>
            </div>
            
            <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
              <div className="bg-white p-4 rounded-lg text-center">
                <Target className="text-blue-500 mx-auto mb-2" size={24} />
                <div className="text-2xl font-bold text-gray-800">{stats.totalQuestions}</div>
                <div className="text-sm text-gray-600">Total de Questões</div>
              </div>
              
              <div className="bg-white p-4 rounded-lg text-center">
                <Clock className="text-green-500 mx-auto mb-2" size={24} />
                <div className="text-2xl font-bold text-gray-800">{stats.avgTime}</div>
                <div className="text-sm text-gray-600">Tempo Médio</div>
              </div>
              
              <div className="bg-white p-4 rounded-lg text-center">
                <TrendingUp className="text-purple-500 mx-auto mb-2" size={24} />
                <div className="text-2xl font-bold text-gray-800">{stats.accuracy}%</div>
                <div className="text-sm text-gray-600">Taxa de Acerto</div>
              </div>
              
              <div className="bg-white p-4 rounded-lg text-center">
                <CheckCircle className="text-orange-500 mx-auto mb-2" size={24} />
                <div className="text-2xl font-bold text-gray-800">{stats.correctAnswers}/{stats.totalAnswered}</div>
                <div className="text-sm text-gray-600">Acertos</div>
              </div>
            </div>
          </div>
        )}

        {/* Lista de Sessões */}
        {showStats && sessions.length > 0 && (
          <div className="bg-white rounded-xl border border-gray-200">
            <h3 className="text-lg font-bold p-4 border-b border-gray-200">Histórico de Questões</h3>
            <div className="max-h-96 overflow-y-auto">
              {sessions.slice().reverse().map((session) => (
                <div key={session.id} className="flex items-center justify-between p-4 border-b border-gray-100 hover:bg-gray-50">
                  <div className="flex-1">
                    <div className="font-semibold text-gray-800">{session.question}</div>
                    <div className="text-sm text-gray-600">{session.timestamp}</div>
                  </div>
                  
                  <div className="text-lg font-mono font-bold text-gray-800 mx-4">
                    {session.formattedTime}
                  </div>
                  
                  <div className="flex items-center gap-2">
                    {session.correct === null ? (
                      <>
                        <button
                          onClick={() => markAnswer(session.id, true)}
                          className="p-2 text-green-600 hover:bg-green-100 rounded-lg transition-colors"
                          title="Marcar como correto"
                        >
                          <CheckCircle size={20} />
                        </button>
                        <button
                          onClick={() => markAnswer(session.id, false)}
                          className="p-2 text-red-600 hover:bg-red-100 rounded-lg transition-colors"
                          title="Marcar como incorreto"
                        >
                          <XCircle size={20} />
                        </button>
                      </>
                    ) : (
                      <div className={`p-2 rounded-lg ${session.correct ? 'bg-green-100 text-green-600' : 'bg-red-100 text-red-600'}`}>
                        {session.correct ? <CheckCircle size={20} /> : <XCircle size={20} />}
                      </div>
                    )}
                    
                    <button
                      onClick={() => deleteSession(session.id)}
                      className="p-2 text-gray-400 hover:text-red-600 hover:bg-red-100 rounded-lg transition-colors ml-2"
                      title="Excluir"
                    >
                      ×
                    </button>
                  </div>
                </div>
              ))}
            </div>
          </div>
        )}

        {/* Instruções para Notion */}
        <div className="mt-8 p-4 bg-blue-50 rounded-lg">
          <h4 className="font-bold text-blue-800 mb-2">Como integrar com o Notion:</h4>
          <ol className="text-sm text-blue-700 space-y-1">
            <li>1. Clique em "Exportar" para baixar os dados em JSON</li>
            <li>2. No Notion, crie uma nova página ou database</li>
            <li>3. Use os dados exportados para criar tabelas com colunas: Questão, Tempo, Data, Acertou</li>
            <li>4. As estatísticas podem ser calculadas usando fórmulas do Notion</li>
          </ol>
        </div>
      </div>
    </div>
  );
};

export default QuestionTimer;

```react
import React, { useState, useEffect, useRef } from 'react';
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from 'firebase/auth';
import { getFirestore, collection, onSnapshot, doc, deleteDoc, addDoc } from 'firebase/firestore';
import * as Icons from 'lucide-react';
import { 
  Github, 
  Linkedin, 
  Mail, 
  Phone,
  ExternalLink, 
  ChevronDown,
  Briefcase,
  Users,
  Target,
  CheckCircle2,
  Settings,
  Plus,
  Trash2,
  X,
  Bell,
  Megaphone,
  Lightbulb,
  PenTool,
  Layers,
  Award
} from 'lucide-react';
import { SpeedInsights } from '@vercel/speed-insights/react';

// إعداد قاعدة بيانات Firebase
const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

// مكون الحركة (أنيميشن)
const Reveal = ({ children, direction = 'up', delay = 0 }) => {
  const [isVisible, setIsVisible] = useState(false);
  const ref = useRef(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsVisible(true);
          observer.unobserve(entry.target);
        }
      },
      { threshold: 0.1 }
    );
    if (ref.current) observer.observe(ref.current);
    return () => observer.disconnect();
  }, []);

  const getTransform = () => {
    if (direction === 'up') return 'translate-y-16';
    if (direction === 'down') return '-translate-y-16';
    if (direction === 'left') return '-translate-x-16'; 
    if (direction === 'right') return 'translate-x-16';
    return 'translate-y-0';
  };

  return (
    <div
      ref={ref}
      className={`transition-all duration-1000 ease-out ${
        isVisible ? 'opacity-100 transform-none' : `opacity-0 ${getTransform()}`
      }`}
      style={{ transitionDelay: `${delay}ms` }}
    >
      {children}
    </div>
  );
};

export default function App() {
  const [activeSection, setActiveSection] = useState('home');
  const [user, setUser] = useState(null);
  const [projectsData, setProjectsData] = useState([]);
  const [newsItems, setNewsItems] = useState([]);
  const [isAdminMode, setIsAdminMode] = useState(false);
  const [showAdminModal, setShowAdminModal] = useState(false);
  const [newProject, setNewProject] = useState({ title: '', description: '', tags: '', iconName: 'Megaphone' });
  const [newNews, setNewNews] = useState('');

  // مصادقة Firebase
  useEffect(() => {
    const initAuth = async () => {
      try {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          await signInWithCustomToken(auth, __initial_auth_token);
        } else {
          await signInAnonymously(auth);
        }
      } catch(e) { console.error("Auth error", e); }
    };
    initAuth();
    const unsubscribe = onAuthStateChanged(auth, setUser);
    return () => unsubscribe();
  }, []);

  // جلب البيانات
  useEffect(() => {
    if (!user) return;
    
    // جلب المشاريع
    const projectsRef = collection(db, 'artifacts', appId, 'public', 'data', 'projects');
    const unsubProjects = onSnapshot(projectsRef, (snapshot) => {
      if (!snapshot.empty) {
        const data = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
        setProjectsData(data.sort((a, b) => b.createdAt - a.createdAt));
      } else setProjectsData([]);
    }, (err) => console.error(err));

    // جلب الأخبار
    const newsRef = collection(db, 'artifacts', appId, 'public', 'data', 'news');
    const unsubNews = onSnapshot(newsRef, (snapshot) => {
      if (!snapshot.empty) {
        const data = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
        setNewsItems(data.sort((a, b) => b.createdAt - a.createdAt));
      } else setNewsItems([]);
    }, (err) => console.error(err));

    return () => { unsubProjects(); unsubNews(); };
  }, [user]);

  // البيانات الافتراضية
  const defaultProjects = [
    { id: 1, title: 'حملات إعلانية متكاملة', description: 'تخطيط وتنفيذ حملات إعلانية رقمية وميدانية لزيادة الوعي.', tags: ['تسويق رقمي', 'إعلانات طرق'], iconName: 'Megaphone' },
    { id: 2, title: 'لوحات ضوئية وإرشادية', description: 'تصميم وتركيب لوحات إعلانية ضوئية (3D, LED, Neon).', tags: ['لوحات LED', 'أحرف بارزة'], iconName: 'Lightbulb' },
    { id: 3, title: 'تصاميم إبداعية', description: 'ابتكار هويات بصرية متكاملة (شعارات، مطبوعات).', tags: ['تصميم جرافيك', 'هوية بصرية'], iconName: 'PenTool' },
    { id: 4, title: 'تنفيذ مشاريع إعلانية', description: 'الإشراف الهندسي وتجهيز المعارض وتكسية الواجهات.', tags: ['تجهيز معارض', 'كلادينج'], iconName: 'Layers' }
  ];

  const defaultNews = [
    { id: 1, text: '🎉 تم افتتاح قسم خاص بتصنيع لوحات النيون 3D بأحدث التقنيات.' },
    { id: 2, text: '🔥 خصم 20% على جميع حملات التسويق الرقمي وحروف الكلادينج هذا الشهر!' }
  ];

  const displayProjects = projectsData.length > 0 ? projectsData : defaultProjects;
  const displayNews = newsItems.length > 0 ? newsItems : defaultNews;

  // دوال لوحة التحكم
  const handleAddProject = async (e) => {
    e.preventDefault();
    if(!user) return;
    try {
      const tagsArray = newProject.tags.split(',').map(t => t.trim()).filter(t => t !== '');
      await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'projects'), {
        title: newProject.title, description: newProject.description, tags: tagsArray, iconName: newProject.iconName, createdAt: Date.now()
      });
      setNewProject({ title: '', description: '', tags: '', iconName: 'Megaphone' });
    } catch(e) { console.error(e); }
  };

  const handleDeleteProject = async (id) => {
    if(!user) return;
    try { await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'projects', id)); } catch(e) { console.error(e); }
  };

  const handleAddNews = async (e) => {
    e.preventDefault();
    if(!user || !newNews) return;
    try {
      await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'news'), { text: newNews, createdAt: Date.now() });
      setNewNews('');
    } catch(e) { console.error(e); }
  };

  const handleDeleteNews = async (id) => {
    if(!user) return;
    try { await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'news', id)); } catch(e) { console.error(e); }
  };

  const handleWhatsAppSubmit = (e) => {
    e.preventDefault();
    const name = e.target.name.value;
    const service = e.target.service.value;
    const details = e.target.details.value;
    if(!name || !details) return; 
    const text = `مرحباً العليمي للإعلان والديكور،\nأنا: ${name}.\nأرغب في الاستفسار عن: ${service}.\nتفاصيل إضافية: ${details}`;
    window.open(`https://wa.me/9672353640?text=${encodeURIComponent(text)}`, '_blank');
  };

  const scrollToSection = (id) => {
    const element = document.getElementById(id);
    if (element) {
      element.scrollIntoView({ behavior: 'smooth' });
      setActiveSection(id);
    }
  };

  const BrandLogo = ({ isFooter = false }) => (
    <div className="flex flex-col items-center justify-center select-none cursor-pointer">
      <span className={`font-black ${isFooter ? 'text-4xl text-white' : 'text-3xl text-[#0B2046]'} tracking-tighter leading-none mb-1`}>العليمي</span>
      <span className={`font-black ${isFooter ? 'text-base' : 'text-sm'} text-[#E85D22] tracking-tight leading-none`}>للإعلان والديكور</span>
      <span className={`font-bold ${isFooter ? 'text-[9px]' : 'text-[7px] md:text-[8px]'} text-[#E85D22] mt-1 tracking-wider uppercase text-center leading-tight`}>AL-Alimi For advertising & decoration</span>
    </div>
  );

  return (
    <div dir="rtl" className="min-h-screen bg-white text-[#0B2046] font-sans selection:bg-[#E85D22]/20 selection:text-[#0B2046]">
      <style>{`
        @keyframes marquee-rtl { 0% { transform: translateX(-100vw); } 100% { transform: translateX(100vw); } }
        .animate-marquee { display: inline-flex; animation: marquee-rtl 35s linear infinite; }
        .ticker-wrapper:hover .animate-marquee { animation-play-state: paused; }
      `}</style>

      {/* Navigation */}
      <nav className="fixed top-0 w-full bg-white/95 backdrop-blur-md shadow-sm z-50 transition-all border-b border-slate-100">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div className="flex justify-between items-center h-24">
            <div className="flex-shrink-0 flex items-center gap-2" onClick={() => scrollToSection('home')}><BrandLogo /></div>
            <div className="hidden lg:flex space-x-8 space-x-reverse">
              {['home', 'about', 'services', 'portfolio', 'experience'].map(sec => (
                <button key={sec} onClick={() => scrollToSection(sec)} className="text-slate-600 hover:text-[#E85D22] font-bold transition-colors">
                  {sec === 'home' ? 'الرئيسية' : sec === 'about' ? 'من نحن' : sec === 'services' ? 'خدماتنا' : sec === 'portfolio' ? 'أعمالنا' : 'مسيرتنا'}
                </button>
              ))}
            </div>
            <div className="hidden lg:flex">
              <button onClick={() => scrollToSection('contact')} className="px-6 py-2.5 bg-[#0B2046] text-white font-medium rounded-full hover:bg-[#E85D22] transition-colors shadow-md">اطلب تسعيرة</button>
            </div>
            <div className="lg:hidden flex items-center">
              <button className="text-[#0B2046] focus:outline-none p-2 bg-slate-50 rounded-lg"><Icons.Menu className="h-6 w-6" /></button>
            </div>
          </div>
        </div>
      </nav>

      {/* News Ticker */}
      <div className="fixed top-24 w-full bg-[#E85D22] text-white h-10 flex items-center z-40 border-b border-white/10 overflow-hidden ticker-wrapper shadow-md">
        <div className="absolute right-0 bg-[#0B2046] h-full px-4 flex items-center gap-2 z-10 border-l border-white/10 shadow-[0_0_15px_#0B2046]">
          <Bell className="w-4 h-4 text-[#E85D22] animate-bounce" />
          <span className="font-bold text-sm tracking-wide">أحدث الأخبار:</span>
        </div>
        <div className="animate-marquee flex items-center gap-16 pr-44 text-sm font-medium">
          {displayNews.map((item, idx) => (
            <span key={item.id || idx} className="flex items-center gap-3 whitespace-nowrap">
              <span className="w-1.5 h-1.5 rounded-full bg-white/60"></span>{item.text}
            </span>
          ))}
        </div>
      </div>

      {/* Hero */}
      <section id="home" className="pt-44 pb-20 md:pt-52 md:pb-28 px-4 flex flex-col items-center justify-center min-h-screen text-center relative overflow-hidden bg-[#0B2046]">
        <div className="absolute inset-0 z-0">
          <div className="absolute inset-0 bg-[url('https://images.unsplash.com/photo-1557804506-669a67965ba0?q=80&w=2074&auto=format&fit=crop')] bg-cover bg-center opacity-30 mix-blend-screen grayscale-[20%]"></div>
          <div className="absolute inset-0 bg-gradient-to-t from-[#0B2046] via-[#0B2046]/80 to-transparent"></div>
          <div className="absolute inset-0 bg-gradient-to-b from-[#0B2046]/80 to-transparent"></div>
        </div>
        <div className="absolute top-1/3 right-1/4 w-[500px] h-[500px] bg-[#E85D22] rounded-full mix-blend-screen filter blur-[150px] opacity-20 animate-pulse z-0"></div>
        <div className="absolute bottom-0 left-1/4 w-[400px] h-[400px] bg-blue-500 rounded-full mix-blend-screen filter blur-[150px] opacity-20 z-0 animate-[pulse_4s_ease-in-out_infinite]"></div>

        <div className="relative z-10 mt-10">
          <Reveal direction="down" delay={100}>
            <div className="inline-flex items-center gap-2 px-5 py-2.5 rounded-full bg-white/10 backdrop-blur-md text-[#E85D22] font-medium text-sm mb-8 border border-[#E85D22]/30 shadow-2xl">
              <Award className="w-5 h-5" /><span className="tracking-wide">خبراء اللوحات الضوئية والكلادينج</span>
            </div>
          </Reveal>
          <Reveal direction="up" delay={200}>
            <h1 className="text-5xl md:text-7xl lg:text-8xl font-black tracking-tight text-white mb-6 leading-tight drop-shadow-2xl">
              نصنع تأثيراً <br/><span className="text-transparent bg-clip-text bg-gradient-to-r from-[#E85D22] to-[#FFB088] filter drop-shadow-lg">يلفت الأنظار لعلامتك</span>
            </h1>
          </Reveal>
          <Reveal direction="up" delay={300}>
            <p className="text-lg md:text-2xl text-slate-300 max-w-3xl mx-auto mb-12 leading-relaxed font-light">
              من التصميم الإبداعي إلى التنفيذ على أرض الواقع. نحن فريق متكامل نقدم لك حلولاً إعلانية وواجهات تبرز هويتك بقوة.
            </p>
          </Reveal>
          <Reveal direction="up" delay={400}>
            <div className="flex flex-col sm:flex-row justify-center gap-5">
              <button onClick={() => scrollToSection('portfolio')} className="px-8 py-4 bg-[#E85D22] text-white font-bold rounded-2xl hover:bg-[#CC4E1B] transition-all duration-300 shadow-[0_0_40px_rgba(232,93,34,0.4)] flex items-center justify-center gap-3 text-lg"><Briefcase className="w-6 h-6" />تصفح أعمالنا</button>
              <button onClick={() => scrollToSection('services')} className="px-8 py-4 bg-white/10 backdrop-blur-md text-white border border-white/20 font-bold rounded-2xl hover:bg-white hover:text-[#0B2046] transition-all duration-300 shadow-xl flex items-center justify-center gap-3 text-lg"><Target className="w-6 h-6" />تعرف على خدماتنا</button>
            </div>
          </Reveal>
        </div>
      </section>

      {/* About */}
      <section id="about" className="py-24 bg-white px-4 overflow-hidden">
        <div className="max-w-6xl mx-auto grid lg:grid-cols-2 gap-16 items-center">
          <Reveal direction="right" delay={200}>
            <div className="relative group">
              <div className="aspect-square bg-[#0B2046] rounded-3xl overflow-hidden relative shadow-2xl transition-transform duration-700 group-hover:scale-[1.02]">
                <div className="absolute inset-0 flex flex-col items-center justify-center bg-[#0B2046] p-8 text-center overflow-hidden">
                  <div className="absolute w-full h-full bg-[radial-gradient(circle_at_center,_var(--tw-gradient-stops))] from-[#112957] to-[#0B2046] opacity-80"></div>
                  <span className="font-black text-6xl md:text-8xl text-white opacity-5 relative z-10 rotate-[-15deg] scale-150">العليمي</span>
                  <Users className="w-32 h-32 text-white/20 relative z-10 mt-8 group-hover:text-[#E85D22]/80 transition-colors duration-700" />
                </div>
              </div>
              <div className="absolute -bottom-6 -left-6 bg-[#E85D22] text-white p-8 rounded-3xl shadow-xl hidden md:block animate-[bounce_3s_infinite]">
                <p className="text-4xl font-extrabold mb-1">+10</p><p className="font-medium opacity-90">سنوات من الخبرة</p>
              </div>
            </div>
          </Reveal>
          <Reveal direction="left" delay={400}>
            <div className="space-y-8">
              <div>
                <h2 className="text-[#E85D22] font-bold tracking-wider uppercase mb-2">من نحن</h2>
                <h3 className="text-3xl md:text-4xl font-bold text-[#0B2046] mb-6">شريكك الاستراتيجي في رحلة النجاح</h3>
              </div>
              <div className="space-y-4 text-slate-600 leading-relaxed text-lg font-medium">
                <p>نحن وكالة إعلانية متكاملة نؤمن بأن التصميم الجيد والتنفيذ المتقن هما لغة العصر.</p>
                <p>سواء كنت تبحث عن إطلاق حملة إعلانية ضخمة، أو تنفيذ لوحات ضوئية، فإن فريقنا جاهز لتحويل أفكارك لواقع.</p>
              </div>
            </div>
          </Reveal>
        </div>
      </section>

      {/* Portfolio */}
      <section id="portfolio" className="py-24 bg-slate-50 px-4 overflow-hidden">
        <div className="max-w-6xl mx-auto">
          <Reveal direction="up" delay={100}>
            <div className="text-center mb-16">
              <h2 className="text-[#E85D22] font-bold tracking-wider uppercase mb-2">أعمالنا السابقة</h2>
              <h3 className="text-3xl md:text-4xl font-bold text-[#0B2046] mb-4">مشاريع نفخر بها</h3>
              <div className="w-24 h-1.5 bg-[#E85D22] mx-auto rounded-full"></div>
            </div>
          </Reveal>
          <div className="grid grid-cols-1 md:grid-cols-2 gap-8">
            {displayProjects.map((project, idx) => {
              const IconComponent = Icons[project.iconName] || Icons.Briefcase;
              return (
                <Reveal key={project.id || idx} direction="up" delay={200 + (idx * 150)}>
                  <div className="bg-white rounded-[2rem] p-8 shadow-sm border border-slate-100 hover:shadow-2xl hover:-translate-y-2 hover:border-[#E85D22]/30 transition-all duration-300 group h-full">
                    <div className="flex items-start justify-between mb-6">
                      <div className="w-20 h-20 bg-[#FFF5F0] text-[#E85D22] rounded-2xl flex items-center justify-center group-hover:bg-[#E85D22] group-hover:text-white group-hover:rotate-[360deg] transition-all duration-700 shadow-inner">
                        <IconComponent className="w-10 h-10" />
                      </div>
                    </div>
                    <h3 className="text-2xl font-bold text-[#0B2046] mb-4">{project.title}</h3>
                    <p className="text-slate-600 mb-8 text-lg leading-relaxed">{project.description}</p>
                  </div>
                </Reveal>
              );
            })}
          </div>
        </div>
      </section>

      {/* Contact */}
      <section id="contact" className="py-24 bg-[#F8FAFC] px-4 overflow-hidden">
        <div className="max-w-6xl mx-auto bg-white rounded-[3rem] p-8 md:p-16 shadow-xl border border-slate-100 grid lg:grid-cols-2 gap-16 items-center">
          <Reveal direction="right" delay={100}>
            <div>
              <h2 className="text-3xl font-bold text-[#0B2046] mb-6">هل لديك فكرة مشروع؟ <br/> <span className="text-[#E85D22]">دعنا ننفذها معاً</span></h2>
              <div className="space-y-6 mt-10">
                <div className="flex items-center gap-6 p-4 bg-[#F8FAFC] rounded-2xl border border-slate-100">
                  <div className="w-14 h-14 bg-white text-[#E85D22] rounded-xl flex items-center justify-center shadow-sm shrink-0"><Phone className="w-7 h-7" /></div>
                  <div className="w-full">
                    <h4 className="font-bold text-[#0B2046] mb-2 text-lg">أرقام التواصل</h4>
                    <div className="space-y-2 w-full">
                      <div className="flex justify-between border-b pb-2"><span className="text-slate-600">الاستعلامات:</span><span className="font-bold" dir="ltr">02353634</span></div>
                      <div className="flex justify-between border-b pb-2"><span className="text-slate-600">المبيعات:</span><span className="font-bold" dir="ltr">778825445</span></div>
                      <div className="flex justify-between border-b pb-2"><span className="text-slate-600">الإدارة:</span><span className="font-bold" dir="ltr">777255955</span></div>
                      <div className="flex justify-between"><span className="text-slate-600">واتس اب:</span><span className="font-bold" dir="ltr">+9672353640</span></div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </Reveal>
          <Reveal direction="left" delay={300}>
            <form className="bg-[#F8FAFC] p-8 md:p-10 rounded-3xl space-y-6 border border-slate-100" onSubmit={handleWhatsAppSubmit}>
              <h3 className="text-2xl font-bold text-[#0B2046]">أرسل رسالة</h3>
              <input type="text" name="name" required className="w-full px-5 py-4 rounded-xl border focus:ring-[#E85D22] outline-none" placeholder="الاسم الكريم" />
              <select name="service" className="w-full px-5 py-4 rounded-xl border focus:ring-[#E85D22] outline-none">
                <option value="حملة إعلانية">حملة إعلانية</option>
                <option value="تصميم وتركيب لوحات ضوئية">تصميم وتركيب لوحات ضوئية</option>
              </select>
              <textarea name="details" required rows="4" className="w-full px-5 py-4 rounded-xl border focus:ring-[#E85D22] outline-none resize-none" placeholder="تفاصيل المشروع..."></textarea>
              <button type="submit" className="w-full py-4 bg-[#0B2046] text-white rounded-xl font-bold hover:bg-[#E85D22] transition-colors">إرسال عبر واتساب</button>
            </form>
          </Reveal>
        </div>
      </section>

      {/* Footer & Admin Modal */}
      <footer className="bg-[#0B2046] text-slate-400 py-16 px-4 relative">
        <div className="max-w-6xl mx-auto flex flex-col md:flex-row justify-between items-center gap-8">
          <div onDoubleClick={() => setIsAdminMode(true)} className="cursor-pointer" title="انقر مزدوجاً لفتح الإعدادات"><BrandLogo isFooter={true} /></div>
          <p>© {new Date().getFullYear()} جميع الحقوق محفوظة.</p>
          {isAdminMode && <button onClick={() => setShowAdminModal(true)} className="p-3 bg-[#E85D22] rounded-full text-white animate-pulse"><Settings className="w-5 h-5" /></button>}
        </div>
      </footer>

      {showAdminModal && (
        <div className="fixed inset-0 bg-black/60 backdrop-blur-sm z-[100] flex items-center justify-center p-4">
          <div className="bg-white rounded-3xl w-full max-w-4xl max-h-[90vh] overflow-y-auto shadow-2xl p-8" dir="rtl">
            <div className="flex justify-between items-center mb-6 border-b pb-4">
              <h2 className="text-2xl font-bold flex items-center gap-2"><Settings className="w-6 h-6 text-[#E85D22]"/>لوحة التحكم</h2>
              <button onClick={() => setShowAdminModal(false)}><X className="w-6 h-6 text-red-500"/></button>
            </div>
            <div className="grid md:grid-cols-2 gap-8">
              <form onSubmit={handleAddProject} className="space-y-4">
                <h3 className="font-bold border-b pb-2">إضافة مشروع</h3>
                <input type="text" required value={newProject.title} onChange={e => setNewProject({...newProject, title: e.target.value})} placeholder="العنوان" className="w-full p-3 border rounded" />
                <textarea required value={newProject.description} onChange={e => setNewProject({...newProject, description: e.target.value})} placeholder="الوصف" className="w-full p-3 border rounded"></textarea>
                <input type="text" required value={newProject.tags} onChange={e => setNewProject({...newProject, tags: e.target.value})} placeholder="كلمات دلالية" className="w-full p-3 border rounded" />
                <button type="submit" className="w-full p-3 bg-[#0B2046] text-white rounded font-bold">إضافة المشروع</button>
              </form>
              <form onSubmit={handleAddNews} className="space-y-4">
                <h3 className="font-bold border-b pb-2">إضافة خبر للشريط</h3>
                <textarea required value={newNews} onChange={e => setNewNews(e.target.value)} placeholder="نص الخبر..." className="w-full p-3 border rounded"></textarea>
                <button type="submit" className="w-full p-3 bg-[#E85D22] text-white rounded font-bold">إضافة الخبر</button>
              </form>
            </div>
          </div>
        </div>
      )}
      <SpeedInsights />
    </div>
  );
}


```
# Anaa

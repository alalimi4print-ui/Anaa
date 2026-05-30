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
  Award,
  Menu
} from 'lucide-react';

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
  const [mobileMenuOpen, setMobileMenuOpen] = useState(false);

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
      setMobileMenuOpen(false); // غلق الموبايل منيو عند اختيار عنصر
    }
  };

  const BrandLogo = ({ isFooter = false }) => (
    <div className="flex flex-col items-center justify-center select-none cursor-pointer">
      <span className={`font-black ${isFooter ? 'text-4xl text-white' : 'text-3xl text-[#0B2046]'} tracking-tighter leading-none mb-1`}>العليمي</span>
      <span className={`font-black ${isFooter ? 'text-base' : 'text-sm'} text-[#E85D22] tracking-tight leading-none`}>للإعلان والديكور</span>
      <span className={`font-bold ${isFooter ? 'text-[9px]' : 'text-[7px] md:text-[8px]'} text-[#E85D22] mt-1 tracking-wider uppercase text-center leading-tight`}>AL-Alimi For advertising & decoration</span>
    </div>
  );

  // قائمة الأيقونات المتاحة لواجهة الإدارة
  const iconOptions = [
    { name: 'Megaphone', label: 'حملة إعلانية' },
    { name: 'Lightbulb', label: 'لوحات ضوئية' },
    { name: 'PenTool', label: 'تصميم' },
    { name: 'Layers', label: 'تنفيذ مشاريع' },
    { name: 'Briefcase', label: 'مشاريع' }
  ];

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
              <button className="text-[#0B2046] focus:outline-none p-2 bg-slate-50 rounded-lg" onClick={() => setMobileMenuOpen(!mobileMenuOpen)}>
                <Menu className="h-6 w-6" />
              </button>
            </div>
          </div>
        </div>
        {/* Mobile Menu */}
        {mobileMenuOpen && (
          <div className="lg:hidden bg-white shadow-lg border-b border-t border-slate-100 absolute w-full z-50 px-6 py-6 animate-fadeIn">
            <div className="flex flex-col gap-5">
              {['home', 'about', 'services', 'portfolio', 'experience'].map(sec => (
                <button key={sec} onClick={() => scrollToSection(sec)} className="text-[#0B2046] font-bold py-2 border-b border-slate-100 text-right">
                  {sec === 'home' ? 'الرئيسية' : sec === 'about' ? 'من نحن' : sec === 'services' ? 'خدماتنا' : sec === 'portfolio' ? 'أعمالنا' : 'مسيرتنا'}
                </button>
              ))}
              <button onClick={() => scrollToSection('contact')} className="w-full px-4 py-3 bg-[#0B2046] text-white font-bold rounded-lg hover:bg-[#E85D22] transition">اطلب تسعيرة</button>
            </div>
          </div>
        )}
      </nav>

      {/* News Ticker */}
      <div className="fixed top-24 w-full bg-[#E85D22] text-white h-10 flex items-center z-40 border-b border-white/10 overflow-hidden ticker-wrapper shadow-md">
        <div className="absolute right-0 bg-[#0B2046] h-full px-4 flex items-center gap-2 z-10 border-l border-white/10 shadow-[0_0_15px_#0B2046]">
          <Bell className="w-4 h-4 text-[#E85D22] animate-bounce" />
          <span className="font-bold text-sm tracking-wide">أحدث الأخبار:</span>
        </div>
        <div className="animate-marquee flex items-center gap-16 pr-44 text-sm font-medium relative">
          {displayNews.map((item, idx) => (
            <span key={item.id || idx} className="flex items-center gap-3 whitespace-nowrap">
              <span className="w-1.5 h-1.5 rounded-full bg-white/60"></span>{item.text}
              {isAdminMode && user && newsItems.some(x=>x.id===item.id) && (
                <button onClick={() => handleDeleteNews(item.id)} className="ml-2 text-red-600 hover:text-red-800 bg-white/20 rounded-full p-1"><Trash2 className="inline w-4 h-4" /></button>
              )}
            </span>
          ))}
        </div>
      </div>

      {/* أكمل باقي الصفحة ... جميع الأكواد كما لديك، فقط 
           - أضف في نموذج المشروع بلوحة الإدارة الحقل التالي بعد عنوان المشروع: */}

      {/* ضمن لوحة الإدارة: إضافة مشروع */}
      {/* ... */}
      {/* <input type="text" required ... /> */}
      <select 
        value={newProject.iconName}
        onChange={e => setNewProject({ ...newProject, iconName: e.target.value })}
        className="w-full p-3 border rounded"
      >
        {iconOptions.map(opt => 
          <option value={opt.name} key={opt.name}>{opt.label}</option>
        )}
      </select>
      {/* <textarea required ... /> */}
      {/* ... */}

      {/* بقية صفحتك كما هي دون تغيير كبير. */}

      {/* يمكنك دائماً نقل هذا القسم في المكان الصحيح داخل الكود الأساسي. */}

    </div>
  );
}

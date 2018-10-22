## 요약
- MVC 패턴과 모델2 구조의 매핑을 통한 회원제 게시판 구현

<br />
<br />

## 목차
>해당 목차는  [원본페이지](https://github.com/zeroam/studynote/wiki/%5BJSP%5D%ED%9A%8C%EC%9B%90%EC%A0%9C-%EA%B2%8C%EC%8B%9C%ED%8C%90-%EA%B5%AC%ED%98%84)에서만 적용 됩니다.

- [환경 설정](#환경-설정)
    + [자바 프로젝트의 web.xml 수정](#자바-프로젝트의-webxml-수정)
    + [commandURI.properties 파일 생성](#commanduriproperties-파일-생성)
- [UML 다이어그램](#uml-다이어그램)
- [MVC 패턴 구현](#mvc-패턴-구현)
  * [컨트롤러(Controller) 영역](#컨트롤러controller-영역)
  * [모델(Model)영역](#모델model영역)
    + [Service 클래스](#service클래스)
      - [service 클래스가 공통적으로 구현해야 할 인터페이스](#service-클래스가-공통적으로-구현해야-할-인터페이스)
      - [로그인 서비스](#로그인-서비스)
      - [로그아웃 서비스](#로그아웃-서비스)
      - [약관 서비스](#약관-서비스)
      - [중복 검사 서비스](#중복-검사-서비스)
      - [회원가입 서비스](#회원가입-서비스)
      - [글 목록 서비스](#글-목록-서비스)
      - [글 보기](#글-보기)
      - [글 쓰기](#글-쓰기)
      - [글 삭제](#글-삭제)
      - [댓글 달기](#댓글-달기)
    + [VO 클래스](#vo-클래스)
      - [MemberVO(회원)](#membervo회원)
      - [BoardVO(게시글)](#boardvo게시글)
      - [TermsVO(약관)](#termsvo약관)
    + [DAO 클래스](#dao-클래스)
      - [MemberDAO(회원)](#memberdao회원)
      - [BoardDAO(게시글)](#boarddao게시글)
    + [유틸도구](#유틸도구)
      - [데이터베이스와 접속하기 위한 클래스](#데이터베이스와-접속하기-위한-클래스)
      - [SQL문](#sql문)
  * [뷰(View)영역](#뷰view영역)
    + [인덱스(index.jsp)](#인덱스indexjsp)
    + [로그인(login.jsp)](#로그인loginjsp)
    + [약관(terms.jsp)](#약관termsjsp-)
    + [회원가입(register.jsp)](#회원가입registerjsp-)
    + [글 목록(list.jsp)](#글-목록listjsp)
    + [글 보기(view.jsp)](#글-보기viewjsp)
    + [글 쓰기(write.jsp)](#글-쓰기writejsp)

<br/>
<br/>

## 환경 설정
- #### 자바 프로젝트의 web.xml 수정
  - 컨트롤러 등록
  ```xml
  <!-- *.do로 오는 요청을 모두 MainController 서블릿으로 전달 -->
  <servlet>
  	<servlet-name>MainController</servlet-name>
  	<servlet-class>kr.co.jboard2.controller.MainController</servlet-class>
  </servlet>
  <servlet-mapping>
  	<servlet-name>MainController</servlet-name>
  	<url-pattern>*.do</url-pattern>
  </servlet-mapping>
  ```
  - 인코딩 필터
  ```xml
  <!-- UTF-8 캐릭터셋 인코딩 필터 등록 -->
  <filter>
  	<filter-name>encodingFilter</filter-name>
  	<filter-class>org.apache.catalina.filters.SetCharacterEncodingFilter</filter-class>
  	<init-param>
  		<param-name>encoding</param-name>
  		<param-value>UTF-8</param-value>
  	</init-param>
  </filter>
  <filter-mapping>
  	<filter-name>encodingFilter</filter-name>
  	<url-pattern>/</url-pattern>
  </filter-mapping>
  ```

<br>

- #### commandURI.properties 파일 생성
  ```properties
  /index.do=kr.co.jboard2.service.IndexService
  /member/terms.do=kr.co.jboard2.service.member.TermsService
  /member/register.do=kr.co.jboard2.service.member.RegistService
  /member/login.do=kr.co.jboard2.service.member.LoginService
  /member/logout.do=kr.co.jboard2.service.member.LogoutService
  /member/checkUser.do=kr.co.jboard2.service.member.CheckService
  /write.do=kr.co.jboard2.service.WriteService
  /list.do=kr.co.jboard2.service.ListService
  /view.do=kr.co.jboard2.service.ViewService
  /delete.do=kr.co.jboard2.service.DeleteService
  /comment.do=kr.co.jboard2.service.CommentService
  ```

<br />
<br />

## UML 다이어그램
![UML_diagram](https://i.imgur.com/ShhTk1c.png)


## MVC 패턴 구현
<hr>

- ### 컨트롤러(Controller) 영역
  ```java
  public class MainController extends HttpServlet{
  	private static final long serialVersionUID = 1L;
  	private Map<String, Object> instances = new HashMap<>();


  	@Override
  	public void init(ServletConfig config) throws ServletException {
  		//컨트롤러 초기화 작업


  		//commandURI.properties 파일경로 추출
  		ServletContext ctx = config.getServletContext();
  		String path = ctx.getRealPath("/WEB-INF") + "/commandURI.properties";


  		//properties 객체 생성
  		Properties prop = new Properties(); // map과 동일한 자료구조 컬렉션
  		FileInputStream fis = null;

  		try {
  			//commandURI.properties 파일과 입력 스트림 연결
  			fis = new FileInputStream(path);

  			//입력스트림으로 commandURI.properties 데이터 읽어 들이기
  			prop.load(fis);
  		} catch(Exception e) {
  			e.printStackTrace();
  		} finally {

  			if(fis != null) {
  				try {
  					fis.close();
  				} catch(Exception e) {
  					e.printStackTrace();
  				}
  			}
  		}

  		//모델클래스 객체를 생성해서 properties에 저장
  		Iterator<?> it = prop.keySet().iterator();

  		while(it.hasNext()) {
  			String k = it.next().toString();
  			String v = prop.getProperty(k);
  			try {
  				//prop 객체에 저장된 문자열 정보를 가지고 해당 패키지에 있는 클래스를 객체로 생성
  				Class<?> obj = Class.forName(v);
  				Object instance = obj.newInstance();

  				instances.put(k, instance);

  			} catch(Exception e) {
  				e.printStackTrace();
  			}

  		}

  	} //init 끝

  	@Override
  	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
  		requestProc(req, resp);
  	}

  	@Override
  	protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
  		requestProc(req, resp);
  	}

  	private void requestProc(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

  		String view = null;

  		// http://localhost:8080/ch18/hello.do
  		String root = req.getContextPath();
  		String uri = req.getRequestURI();
  		String command = uri.substring(root.length());

  		CommandAction instance = (CommandAction)instances.get(command);
  		view = instance.requestProc(req, resp);

  		if(view.startsWith("redirect:")) {
  			String action = view.substring("redirect:".length());
  			resp.sendRedirect(action);
  		} else if(view.startsWith("json:")) {
  			//json:{'result':'1'}
  			String json = view.substring("json:".length());
  			PrintWriter out = resp.getWriter();
  			out.print(json);
  		} else {
  			RequestDispatcher dispatcher = req.getRequestDispatcher(view);
  			dispatcher.forward(req, resp);			
  		}
  	}
  }
  ```

<br/>

### 모델(Model)영역

- #### Service 클래스
  - ##### service 클래스가 공통적으로 구현해야 할 인터페이스
    ```java
    public interface CommandAction {

    	public String requestProc(HttpServletRequest req, HttpServletResponse resp);

    }
    ```
  - ##### 로그인 서비스
    ```java
    public class LoginService implements CommandAction  {

    	@Override
    	public String requestProc(HttpServletRequest req, HttpServletResponse resp) {

    		String method = req.getMethod();
    		HttpSession session = req.getSession();
    		//전송 방식이 POST 방식일 때
    		if(method.equals("POST")) {
    			//id, pw값 파라미터로 받기
    			String id = req.getParameter("id");
    			String pw = req.getParameter("pw");

    			MemberVO vo = null;
    			MemberDAO memberDAO = MemberDAO.getInstance();
    			vo = memberDAO.login(id, pw);
    			if(vo != null) {
    				//세션값에 멤버 객체 넣기
    				session.setAttribute("member", vo);
    				//로그인 여부
    				session.setAttribute("login", true);
    				return "redirect:/jboard2/list.do";
    			} else {
    				return "redirect:/jboard2/member/login.do?result=fail";				
    			}
    		//세션 값이 있으면 로그인 상태 유지
    		} else if(session.getAttribute("member") != null) {
    			return "redirect:/jboard2/list.do";
    		} else {
    			String result = req.getParameter("result");
    			req.setAttribute("result", result);
    			return "/login.jsp";
    		}
    	}
    }
    ```
  - ##### 로그아웃 서비스
    ```java
    public class LogoutService implements CommandAction{

    	@Override
    	public String requestProc(HttpServletRequest req, HttpServletResponse resp) {
    		//request 객체로 부터 세션값 가져오기
    		HttpSession session = req.getSession();
    		//세션 만료
    		session.invalidate();
    		return "redirect:/jboard2/member/login.do";
    	}
    }  
    ```
  - ##### 약관 서비스
    ```java
    public class TermsService implements CommandAction {

    	@Override
    	public String requestProc(HttpServletRequest req, HttpServletResponse resp) {

    		MemberDAO dao = MemberDAO.getInstance();
    		TermsVO termsVO = dao.terms();
    		//request 객체에 속성값 부여
    		req.setAttribute("vo", termsVO);		

    		return "/terms.jsp";
    	}
    }
    ```
  - ##### 중복 검사 서비스
    ```java
    public class CheckService implements CommandAction {

    	@Override
    	public String requestProc(HttpServletRequest req, HttpServletResponse resp) {

    		String check = req.getParameter("check");
    		String value = req.getParameter("value");

    		MemberDAO dao = MemberDAO.getInstance();

    		int result = 0;

    		result = dao.check(check, value);

    		//json 포맷 문자열 생성
    		String json = "{\"result\":"+result+"}";

    		return "json:"+json;
    	}
    }
    ```
  - ##### 회원가입 서비스
    ```java
    public class RegistService implements CommandAction  {

    	@Override
    	public String requestProc(HttpServletRequest req, HttpServletResponse resp) {

    		String method = req.getMethod();

    		if(method.equals("GET")) {
    			//terms에서 회원가입 페이지로 넘어갈 경우
    			return "/register.jsp";			
    		} else {
    			//register 페이지에서 회원가입을 할 경우
    			boolean result = false;

    			MemberDAO memberDAO = MemberDAO.getInstance();
    			try {
    				MemberVO memberVO = new MemberVO(req);
    				//데이터베이스에 회원 등록이 성공적으로 되었을 때, true값 반환
    				result = memberDAO.register(memberVO);
    			} catch (Exception e) {
    				e.printStackTrace();
    			}

    			if(result) {
    				//회원 등록이 정상적으로 되었을 때
    				return "redirect:/jboard2/member/login.do";				
    			} else {
    				//회원 등록에 실패했을 때
    				return "/register.jsp";
    			}
    		}
    	}
    }
    ```
  - ##### 글 목록 서비스
    ```java
    public class ListService implements CommandAction {

    	@Override
    	public String requestProc(HttpServletRequest req, HttpServletResponse resp) {
    		HttpSession session = req.getSession();
    		//세션값이 없음에도 URL로 접근하는 것 막기
    		if(session.getAttribute("member") == null) {
    			return "redirect:/jboard2/member/login.do";
    		}

    		//현재 페이지
    		int page = 1;
    		if(req.getParameter("page") != null) {
    			page = Integer.parseInt(req.getParameter("page"));			
    		}

    		//arraylist로 boardVO 객체 받기
    		BoardDAO dao = BoardDAO.getInstance();
    		List<BoardVO> voList = dao.list(page);
    		req.setAttribute("list", voList);

    		//전체 페이지 얻고 리퀘스트 객체에 속성 지정하기
    		int total_page = dao.getTotalPage();
    		req.setAttribute("total_page", total_page);

    		//게시글 시작번호 구하기
    		int startNum = dao.getBoardNumber(page);
    		req.setAttribute("startNum", startNum);

    		//현재 페이지 번호 설정
    		req.setAttribute("page", page);

    		//이전 이후 페이지 설정
    		int prepage = 1;
    		if(page > 1) {
    			prepage = page-1;
    		}
    		int postpage = page+1;
    		if(page >= total_page) {
    			postpage = page;
    		}

    		req.setAttribute("prepage", prepage);
    		req.setAttribute("postpage", postpage);

    		//페이지 그룹 계산
    		int groupCurrent = (int)Math.ceil(page/10.0);
    		int groupStart = (groupCurrent-1)*10 + 1;
    		int groupEnd = groupCurrent*10;
    		if(groupEnd > total_page) {
    			groupEnd = total_page;
    		}

    		req.setAttribute("groupStart", groupStart);
    		req.setAttribute("groupEnd", groupEnd);


    		return "/list.jsp";
    	}
    }
    ```
  - ##### 글 보기
    ```java
    public class ViewService implements CommandAction {

    	@Override
    	public String requestProc(HttpServletRequest req, HttpServletResponse resp) {
    		String seq = req.getParameter("seq");
    		String pg = req.getParameter("page");

    		//게시글 vo 객체 얻기
    		BoardDAO dao = BoardDAO.getInstance();
    		BoardVO vo = dao.view(seq);

    		//댓글 리스트 얻기
    		int parent = Integer.parseInt(seq);
    		List<BoardVO> comments = dao.commentList(parent);

    		//리퀘스트 속성값 부여
    		req.setAttribute("vo", vo);
    		req.setAttribute("page", pg);
    		req.setAttribute("comments", comments);

    		return "/view.jsp";
    	}
    }
    ```
  - ##### 글 쓰기
    ```java
    public class WriteService implements CommandAction {


    	@Override
    	public String requestProc(HttpServletRequest req, HttpServletResponse resp) {

    		String method = req.getMethod();

    		//작성완료 버튼을 눌렀을 때
    		if(method.equals("POST")) {
    			//post 값 인코딩
    			try {
    				req.setCharacterEncoding("UTF-8");
    			} catch (UnsupportedEncodingException e) {
    				e.printStackTrace();
    			}

    			//게시판 vo 객체 생성 및 초기화		
    			BoardVO vo = new BoardVO();
    			vo.setCate(req.getParameter("cate"));
    			vo.setTitle(req.getParameter("subject"));
    			vo.setContents(req.getParameter("content"));
    			vo.setUid(req.getParameter("uid"));
    			vo.setRegip(req.getRemoteAddr());

    			//데이터 베이스에 글 등록
    			BoardDAO dao = BoardDAO.getInstance();
    			dao.write(vo);

    			return "redirect:/jboard2/list.do";
    		} else {
    			return "/write.jsp";				
    		}
    	}
    }
    ```
  - ##### 글 삭제
    ```java
    public class DeleteService implements CommandAction {

    	@Override
    	public String requestProc(HttpServletRequest req, HttpServletResponse resp) {
    		//파라미터로 글 번호 값 가져옴
    		String seq = req.getParameter("seq");
    		if(seq != null) {
    			BoardDAO dao = BoardDAO.getInstance();
    			//dao 객체의 delete 실행
    			dao.delete(seq);

    			return "redirect:/jboard2/list.do";
    		} else {
    			return "/jboard2/view.do";
    		}

    	}
    }
    ```
  - ##### 댓글 달기
    ```java
    public class CommentService implements CommandAction {

    	@Override
    	public String requestProc(HttpServletRequest req, HttpServletResponse resp) {
    		try {
    			req.setCharacterEncoding("UTF-8");
    		} catch (UnsupportedEncodingException e) {
    			e.printStackTrace();
    		}


    		//현재 페이지
    		String page = req.getParameter("page");
    		//게시글 번호
    		String parent = req.getParameter("parent");
    		//글 유형
    		String cate = req.getParameter("cate");
    		//댓글 작성자 아이디
    		String uid = req.getParameter("uid");
    		//댓글 작성 내용
    		String comment = req.getParameter("comment");


    		//게시글 vo 객체 생성 및 초기화
    		BoardVO vo = new BoardVO();
    		vo.setParent(Integer.parseInt(parent));
    		vo.setCate(cate);
    		vo.setContents(comment);
    		vo.setUid(uid);
    		vo.setRegip(req.getRemoteAddr());

    		BoardDAO dao = BoardDAO.getInstance();
    		dao.commentWrite(vo);

    		//현재 댓글을 작성한 페이지로 리다이렉트
    		return "redirect:/jboard2/view.do?seq="+parent+"&page="+page;
    	}
    }
    ```

- #### VO 클래스
  - ##### MemberVO(회원)
    ```java
    public class MemberVO {

    	private int seq;
    	private String uid;
    	private String pass;
    	private String name;
    	private String nick;
    	private String email;
    	private String hp;
    	private int grade;
    	private String zip;
    	private String addr1;
    	private String addr2;
    	private String regip;
    	private String rdate;

    	public MemberVO() {

    	}

    	public MemberVO(HttpServletRequest req) throws Exception {
    		req.setCharacterEncoding("UTF-8");
    		this.uid = req.getParameter("id");
    		this.pass = req.getParameter("pw1");
    		this.name = req.getParameter("name");
    		this.nick = req.getParameter("nick");
    		this.email = req.getParameter("email");
    		this.hp = req.getParameter("hp");
    		this.grade = 2;
    		this.zip = req.getParameter("zip");
    		this.addr1 = req.getParameter("addr1");
    		this.addr2 = req.getParameter("addr2");
    		this.regip = req.getRemoteAddr();
    	}

    	//get, set 메소드 ...
    }      
    ```
  - ##### BoardVO(게시글)
    ```java
    public class BoardVO {

    	private int seq;
    	private int parent;
    	private int comment;
    	private String cate;
    	private String title;
    	private String contents;
    	private int file;
    	private int hit;
    	private String uid;
    	private String regip;
    	private String rdate;
    	private String nick;

    	public BoardVO() {

    	}

    	public BoardVO(ResultSet rs) throws SQLException {
    		this.seq = rs.getInt(1);
    		this.parent = rs.getInt(2);
    		this.comment = rs.getInt(3);
    		this.cate = rs.getString(4);
    		this.title = rs.getString(5);
    		this.contents = rs.getString(6);
    		this.file = rs.getInt(7);
    		this.hit = rs.getInt(8);
    		this.uid = rs.getString(9);
    		this.regip = rs.getString(10);
    		this.rdate = rs.getString(11);
    	}

    	//get,set 메소드...
    }
    ```
  - ##### TermsVO(약관)
    ```java
    public class TermsVO {

    	private String terms;
    	private String privacy;

    	public String getTerms() {
    		return terms;
    	}
    	public void setTerms(String terms) {
    		this.terms = terms;
    	}
    	public String getPrivacy() {
    		return privacy;
    	}
    	public void setPrivacy(String privacy) {
    		this.privacy = privacy;
    	}
    }
    ```

- #### DAO 클래스
  - ##### MemberDAO(회원)
    ```java
    public class MemberDAO {

    	// 싱글톤 객체로
    	private static MemberDAO dao = new MemberDAO();

    	public static MemberDAO getInstance() {
    		return dao;
    	}

    	private MemberDAO() {
    	}

    	private Connection conn = null;
    	private Statement stmt = null;
    	private PreparedStatement psmt = null;
    	private ResultSet rs = null;

    	//id, pass값을 가진 데이터 불러오기
    	public MemberVO login(String id, String pw) {
    		MemberVO vo = null;
    		try {
    			conn = DBConfig.getConnection();

    			psmt = conn.prepareStatement(SQL.SELECT_UID_PASS);
    			psmt.setString(1, id);
    			psmt.setString(2, pw);

    			rs = psmt.executeQuery();

    			if(rs.next()) {
    				vo = new MemberVO();
    				vo.setSeq(rs.getInt(1));
    				vo.setUid(rs.getString(2));
    				vo.setPass(rs.getString(3));
    				vo.setName(rs.getString(4));
    				vo.setNick(rs.getString(5));
    				vo.setEmail(rs.getString(6));
    				vo.setHp(rs.getString(7));
    				vo.setGrade(rs.getInt(8));
    				vo.setZip(rs.getString(9));
    				vo.setAddr1(rs.getString(10));
    				vo.setAddr2(rs.getString(11));
    				vo.setRegip(rs.getString(12));
    				vo.setRdate(rs.getString(13));
    			}
    		} catch(SQLException | ClassNotFoundException e) {
    			e.printStackTrace();
    		} finally {
    			DBConfig.close(rs);
    			DBConfig.close(psmt);
    			DBConfig.close(conn);
    		}

    		return vo;
    	}

    	//약관 페이지
    	public TermsVO terms() {

    		String terms = null;
    		String privacy = null;
    		TermsVO termsVO = new TermsVO();
    		try {
    			conn = DBConfig.getConnection();

    			stmt = conn.createStatement();

    			rs = stmt.executeQuery(SQL.SELECT_TERMS);

    			if (rs.next()) {
    				terms = rs.getString(1);
    				privacy = rs.getString(2);

    			}

    			termsVO.setTerms(terms);
    			termsVO.setPrivacy(privacy);

    		} catch (ClassNotFoundException | SQLException e) {
    			e.printStackTrace();
    		} finally {
    			DBConfig.close(rs);
    			DBConfig.close(stmt);
    			DBConfig.close(conn);
    		}
    		return termsVO;
    	}

    	//회원가입 처리
    	public boolean register(MemberVO memberVO) {

    		boolean result = false;

    		try {
    			conn = DBConfig.getConnection();

    			psmt = conn.prepareStatement(SQL.INSERT_MEMBER);
    			psmt.setString(1, memberVO.getUid());
    			psmt.setString(2, memberVO.getPass());
    			psmt.setString(3, memberVO.getName());
    			psmt.setString(4, memberVO.getNick());
    			psmt.setString(5, memberVO.getEmail());
    			psmt.setString(6, memberVO.getHp());
    			psmt.setInt(7, memberVO.getGrade());
    			psmt.setString(8, memberVO.getZip());
    			psmt.setString(9, memberVO.getAddr1());
    			psmt.setString(10, memberVO.getAddr2());
    			psmt.setString(11, memberVO.getRegip());

    			psmt.executeUpdate();

    			result = true;
    		} catch (ClassNotFoundException | SQLException e) {
    			e.printStackTrace();
    		} finally {
    			DBConfig.close(psmt);
    			DBConfig.close(conn);
    		}

    		return result;
    	}

    	//중복 여부 검사
    	public int check(String check, String value) {
    		int result = 0;


    		try {
    			conn = DBConfig.getConnection();

    			switch(check) {
    			case "uid":
    				psmt = conn.prepareStatement(SQL.SELECT_CHECK_ID);
    				break;
    			case "nick":
    				psmt = conn.prepareStatement(SQL.SELECT_CHECK_NICK);
    				break;
    			case "hp":
    				psmt = conn.prepareStatement(SQL.SELECT_CHECK_HP);
    				break;
    			case "email":
    				psmt = conn.prepareStatement(SQL.SELECT_CHECK_EMAIL);
    				break;
    			default:
    				throw new SQLException();
    			}
    			psmt.setString(1, value);

    			rs = psmt.executeQuery();

    			if (rs.next()) {
    				result = rs.getInt(1);
    			}
    		} catch (ClassNotFoundException | SQLException e) {
    			e.printStackTrace();
    		} finally {
    			DBConfig.close(rs);
    			DBConfig.close(stmt);
    			DBConfig.close(conn);
    		}
    		return result;
    	}
    }
    ```
  - ##### BoardDAO(게시글)
    ```java
    public class BoardDAO {
    	// 싱글톤 객체
    	private static BoardDAO dao = new BoardDAO();

    	public static BoardDAO getInstance() {
    		return dao;
    	}

    	private BoardDAO() {
    	}

    	// 드라이버 준비
    	private Connection conn = null;
    	private Statement stmt = null;
    	private PreparedStatement psmt = null;
    	private ResultSet rs = null;

    	// 글 목록 메소드
    	public ArrayList<BoardVO> list(int page) {
    		// 리스트 배열
    		ArrayList<BoardVO> voList = new ArrayList<>();
    		BoardVO vo = null;
    		try {
    			// 게시글 출력 시작할 번호
    			int num = (page - 1) * 10;

    			conn = DBConfig.getConnection();

    			psmt = conn.prepareStatement(SQL.SELECT_BOARD);
    			psmt.setInt(1, num);

    			rs = psmt.executeQuery();

    			while (rs.next()) {
    				// 게시판 객체 초기화
    				vo = new BoardVO(rs);
    				// JOIN 문으로 얻은 nick값 넣기
    				vo.setNick(rs.getString(12));
    				// 게시판 객체 리스트 배열에 추가하기
    				voList.add(vo);
    			}
    		} catch (ClassNotFoundException | SQLException e) {
    			e.printStackTrace();
    		} finally {
    			DBConfig.close(rs);
    			DBConfig.close(stmt);
    			DBConfig.close(conn);
    		}
    		return voList;
    	}

    	// 글쓰기 메소드
    	public void write(BoardVO vo) {
    		try {
    			conn = DBConfig.getConnection();

    			psmt = conn.prepareStatement(SQL.INSERT_BOARD);
    			psmt.setString(1, vo.getCate());
    			psmt.setString(2, vo.getTitle());
    			psmt.setString(3, vo.getContents());
    			psmt.setString(4, vo.getUid());
    			psmt.setString(5, vo.getRegip());

    			psmt.executeUpdate();

    		} catch (SQLException | ClassNotFoundException e) {
    			e.printStackTrace();
    		} finally {
    			DBConfig.close(psmt);
    			DBConfig.close(conn);
    		}

    	}

    	// 글보기 메소드
    	public BoardVO view(String seq) {
    		BoardVO vo = null;

    		try {
    			conn = DBConfig.getConnection();

    			psmt = conn.prepareStatement(SQL.VIEW_BOARD);
    			psmt.setString(1, seq);

    			rs = psmt.executeQuery();

    			if (rs.next()) {
    				vo = new BoardVO(rs);
    			}

    		} catch (SQLException | ClassNotFoundException e) {
    			e.printStackTrace();
    		} finally {
    			DBConfig.close(rs);
    			DBConfig.close(psmt);
    			DBConfig.close(conn);
    		}

    		return vo;
    	}

    	// 글수정 메소드
    	public void modify() {
    	}

    	// 글삭제 메소드
    	public void delete(String seq) {
    		try {
    			conn = DBConfig.getConnection();

    			psmt = conn.prepareStatement(SQL.DELETE_BOARD);
    			psmt.setString(1, seq);

    			psmt.executeUpdate();
    		} catch (SQLException | ClassNotFoundException e) {
    			e.printStackTrace();
    		} finally {
    			DBConfig.close(psmt);
    			DBConfig.close(conn);
    		}

    	}

    	// 댓글 작성 메소드
    	public void commentWrite(BoardVO vo) {
    		try {
    			conn = DBConfig.getConnection();

    			psmt = conn.prepareStatement(SQL.INSERT_COMMENT);
    			psmt.setInt(1, vo.getParent());
    			psmt.setString(2, vo.getCate());
    			psmt.setString(3, vo.getContents());
    			psmt.setString(4, vo.getUid());
    			psmt.setString(5, vo.getRegip());

    			psmt.executeUpdate();
    		} catch (SQLException | ClassNotFoundException e) {

    		} finally {
    			DBConfig.close(psmt);
    			DBConfig.close(conn);
    		}

    	}

    	// 댓글 목록 메소드
    	public ArrayList<BoardVO> commentList(int parent) {
    		// 리스트 배열
    		ArrayList<BoardVO> voList = new ArrayList<>();
    		BoardVO vo = null;
    		try {

    			conn = DBConfig.getConnection();

    			psmt = conn.prepareStatement(SQL.SELECT_COMMENT);
    			psmt.setInt(1, parent);

    			rs = psmt.executeQuery();

    			while (rs.next()) {
    				// 게시판 객체 초기화
    				vo = new BoardVO(rs);
    				// 게시판 객체 리스트 배열에 추가하기
    				voList.add(vo);
    			}
    		} catch (ClassNotFoundException | SQLException e) {
    			e.printStackTrace();
    		} finally {
    			DBConfig.close(rs);
    			DBConfig.close(stmt);
    			DBConfig.close(conn);
    		}
    		return voList;
    	}

    	// 전체 페이지 계산
    	public int getTotalPage() {
    		// 전체 페이지
    		int total_page = 1;

    		try {
    			conn = DBConfig.getConnection();

    			psmt = conn.prepareStatement(SQL.GET_TOTAL_BOARD);

    			rs = psmt.executeQuery();

    			if (rs.next()) {
    				// 전체 게시글 숫자
    				int total_board = rs.getInt(1);

    				// 10으로 나누어 떨어질 경우
    				if (total_board % 10 == 0) {
    					total_page = total_board / 10;
    					// 10으로 안나누어 떨어질 경우
    				} else {
    					total_page = total_board / 10 + 1;
    				}
    			}
    		} catch (SQLException | ClassNotFoundException e) {
    			e.printStackTrace();
    		} finally {
    			DBConfig.close(rs);
    			DBConfig.close(psmt);
    			DBConfig.close(conn);
    		}
    		return total_page;
    	}

    	// 게시글 시작 번호 구하기
    	public int getBoardNumber(int page) {
    		int startNum = 1;

    		try {
    			conn = DBConfig.getConnection();

    			psmt = conn.prepareStatement(SQL.GET_TOTAL_BOARD);

    			rs = psmt.executeQuery();

    			if (rs.next()) {
    				// 전체 게시글 숫자
    				int total_board = rs.getInt(1);
    				startNum = total_board - (page - 1) * 10;
    			}
    		} catch (SQLException | ClassNotFoundException e) {
    			e.printStackTrace();
    		} finally {
    			DBConfig.close(rs);
    			DBConfig.close(psmt);
    			DBConfig.close(conn);
    		}
    		return startNum;
    	}
    }

    ```

- #### 유틸도구
  - ##### 데이터베이스와 접속하기 위한 클래스
    ```java
    public class DBConfig {

    	//데이터 베이스의 주소, 아이디, 비밀번호
    	private static final String HOST = "jdbc:mysql://192.168.0.23:3306/jcw";
    	private static final String USER = "jcw";
    	private static final String PASS = "1234";

    	public static Connection getConnection() throws ClassNotFoundException, SQLException {
    		//1단계 : 드라이버 불러오기
    		Class.forName("com.mysql.jdbc.Driver");
    		//2단계 : 연결 객체 만들기
    		Connection conn = DriverManager.getConnection(HOST, USER, PASS);
    		return conn;
    	}

    	//연결된 접속 끊기
    	public static void close(Connection conn) {
    		if(conn != null) try {conn.close();} catch (SQLException e) {}
    	}
    	public static void close(PreparedStatement psmt) {
    		if(psmt != null) try {psmt.close();} catch (SQLException e) {}
    	}
    	public static void close(Statement stmt) {
    		if(stmt != null) try {stmt.close();} catch (SQLException e) {}
    	}
    	public static void close(ResultSet rs) {
    		if(rs != null) try {rs.close();} catch (SQLException e) {}
    	}
    }
    ```
  - ##### SQL문
    ```java
    public class SQL {

    	public static final String SELECT_TERMS = "SELECT * FROM JB_TERMS";
    	public static final String SELECT_CHECK_ID = "SELECT COUNT(*) FROM JB_MEMBER WHERE uid=?";
    	public static final String SELECT_CHECK_NICK = "SELECT COUNT(*) FROM JB_MEMBER WHERE nick=?";
    	public static final String SELECT_CHECK_HP = "SELECT COUNT(*) FROM JB_MEMBER WHERE hp=?";
    	public static final String SELECT_CHECK_EMAIL = "SELECT COUNT(*) FROM JB_MEMBER WHERE email=?";
    	public static final String SELECT_UID_PASS = "SELECT * FROM JB_MEMBER WHERE uid=? AND pass=?";
    	public static final String INSERT_MEMBER = "INSERT INTO JB_MEMBER (uid, pass, name, nick, email, hp, grade, zip, addr1, addr2, regip, rdate) VALUES (?,?,?,?,?,?,?,?,?,?,?,NOW())";
    	public static final String INSERT_BOARD = "INSERT INTO JB_BOARD (cate, title, contents, uid, regip, rdate) VALUES (?,?,?,?,?,NOW());";
    	public static final String SELECT_BOARD = "SELECT b.*, m.nick FROM JB_BOARD AS b LEFT JOIN JB_MEMBER AS m ON b.uid = m.uid WHERE parent=0 ORDER BY b.seq DESC LIMIT ?, 10;";
    	public static final String VIEW_BOARD = "SELECT * FROM JB_BOARD WHERE seq=?;";
    	public static final String GET_TOTAL_BOARD = "SELECT COUNT(*) FROM JB_BOARD WHERE parent=0;";
    	public static final String DELETE_BOARD = "DELETE FROM JB_BOARD WHERE seq=?";
    	public static final String INSERT_COMMENT ="INSERT INTO JB_BOARD SET parent=?, cate=?, contents=?, uid=?, regip=?, rdate=NOW();";
    	public static final String SELECT_COMMENT = "SELECT * FROM JB_BOARD WHERE parent=?";

    }
    ```

### 뷰(View)영역
- #### 인덱스(index.jsp)
  ```html
  <%@ page contentType="text/html;charset=UTF-8" pageEncoding="UTF-8"%>
  <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
  <% response.sendRedirect("/jboard2/index.do"); %>
  ```
- #### 로그인(login.jsp)
    ```html
    <%@ page contentType="text/html;charset=UTF-8" pageEncoding="UTF-8"%>
    <!DOCTYPE html>
    <html>
    	<head>
    		<meta charset="UTF-8">
    		<title>로그인</title>
    		<link rel="stylesheet" href="/jboard2/css/style.css" />
    		<script>
    			var result = "${result}";

    			if(result == "fail") {
    				alert("아이디와 패스워드를 확인하십시오");
    			}
    		</script>
    	</head>
    	<body>
    		<div id="member">
    			<section class="login">		
    				<form action="/jboard2/member/login.do" method="post">
    					<table>
    						<tr>
    							<td><img src="/jboard2/img/login_ico_id.png" alt="아이디" /></td>
    							<td><input type="text" name="id" required placeholder="아이디 입력" autocomplete="off" /></td>
    						</tr>
    						<tr>
    							<td><img src="/jboard2/img/login_ico_pw.png" alt="비밀번호" /></td>
    							<td><input type="password" name="pw" required placeholder="비밀번호 입력" /></td>
    						</tr>
    					</table>
    					<input type="submit" class="btnLogin" value="로그인" />
    				</form>			

    				<div class="info">
    					<h3>회원로그인 안내</h3>
    					<p>아직 회원이 아니시면 회원으로 가입하세요.</p>
    					<div><a href="/jboard2/member/terms.do">회원가입</a></div>
    				</div>			
    			</section>
    		</div>
    	</body>
    </html>
    ```
- #### 약관(terms.jsp)
  ```html
  <%@ page contentType="text/html;charset=UTF-8" pageEncoding="UTF-8"%>
  <!DOCTYPE html>
  <html lang="en">
  	<head>
  		<meta charset="utf-8">
  		<title>terms</title>
  		<link rel="stylesheet" href="/jboard2/css/style.css" />
  		<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
  		<script src="/jboard2/js/terms.js"></script>
  	</head>

  	<body>
  		<div id="terms">
  			<section>
  				<table>
  					<caption>사이트 이용약관</caption>
  					<tr>
  						<td>
  							<textarea readonly>${ vo.terms }</textarea>
  							<div>
  								<label><input type="checkbox" name="chk1" />&nbsp;동의합니다.</label>        
  							</div>
  						</td>
  					</tr>
  				</table>
  			</section>			
  			<section>
  				<table>
  					<caption>개인정보 취급방침</caption>
  					<tr>
  						<td>
  							<textarea readonly>${ vo.privacy }</textarea>
  							<div>
  								<label><input type="checkbox" name="chk2" />&nbsp;동의합니다.</label>        
  							</div>
  						</td>
  					</tr>
  				</table>
  			</section>

  			<div>
  				<a href="/jboard2/member/login.do" class="btnCancel">취소</a>
  				<a href="/jboard2/member/register.do" class="btnNext">다음</a>
  			</div>
  		</div>
  	</body>
  </html>
  ```
- #### 회원가입(register.jsp)
  ```html
  <%@ page contentType="text/html;charset=UTF-8" pageEncoding="UTF-8"%>
  <!DOCTYPE html>
  <html>
  <head>
  <meta charset="UTF-8">
  <title>회원가입</title>
  <link rel="stylesheet" href="/jboard2/css/style.css" />
  <script	src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
  <script src="/jboard2/js/validation.js"></script>
  <script src="/jboard2/js/check.js"></script>
  <script src="http://dmaps.daum.net/map_js_init/postcode.v2.js"></script>
  <script src="/jboard2/js/zipcode.js"></script>

  </head>
  <body>
  	<div id="member">
  		<section class="register">
  			<form action="/jboard2/member/register.do" method="POST">
  				<section>
  					<table>
  						<caption>사이트 이용정보 입력</caption>
  						<tr>
  							<td>아이디</td>
  							<td><input type="text" name="id" placeholder="아이디를 입력"
  								required
  							/> <span class="resultId"></span></td>
  						</tr>
  						<tr>
  							<td>비밀번호</td>
  							<td><input type="password" name="pw1" placeholder="비밀번호를 입력"
  								required
  							/></td>
  						</tr>
  						<tr>
  							<td>비밀번호확인</td>
  							<td><input type="password" name="pw2" placeholder="비밀번호를 확인"
  								required
  							/> <span class="resultPw"></span></td>
  						</tr>
  					</table>
  				</section>
  				<section>
  					<table>
  						<caption>개인정보 입력</caption>
  						<tr>
  							<td>이름</td>
  							<td><input type="text" name="name" placeholder="이름을 입력"
  								required
  							/></td>
  						</tr>
  						<tr>
  							<td>별명</td>
  							<td><span class="info">공백없이 한글, 영문, 숫자만 입력가능</span>
  								<div>
  									<input type="text" name="nick" placeholder="별명을 입력" required />
  								</div> <span class="resultNick"></span></td>
  						</tr>
  						<tr>
  							<td>EMAIL</td>
  							<td><input type="email" name="email" placeholder="이메일을 입력"
  								required
  							/> <span class="resultEmail"></span></td>
  						</tr>
  						<tr>
  							<td>휴대폰</td>
  							<td><input type="text" name="hp" placeholder="-포함 13자리를 입력"
  								maxlength="13" required
  							/> <span class="resultHp"></span></td>
  						</tr>
  						<tr>
  							<td>주소</td>
  							<td>
  								<div>
  									<input type="text" name="zip" id="zip" placeholder="우편번호"
  										readonly
  									/>
  									<button type="button" class="btnFind" onclick="zipcode()">주소검색</button>
  								</div>
  								<div>
  									<input type="text" name="addr1" id="addr1" size="50"
  										placeholder="주소를 검색하세요." readonly
  									/>
  								</div>
  								<div>
  									<input type="text" name="addr2" id="addr2" size="50"
  										placeholder="상세주소를 입력하세요."
  									/>
  								</div>
  							</td>
  						</tr>
  					</table>
  				</section>
  				<div>
  					<a href="/jboard2/member/login.do" class="cancel">취소</a> <input
  						type="submit" class="join" value="회원가입"
  					/>
  				</div>
  			</form>
  		</section>
  	</div>
  </body>
  </html>
  ```
- #### 글 목록(list.jsp)
  ```html
  <%@ page contentType="text/html;charset=UTF-8" pageEncoding="UTF-8"%>
  <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
  <script>var login = Boolean("${login}");</script>
  <!DOCTYPE html>
  <html>
  	<head>
  		<meta charset="UTF-8" />
  		<title>글목록</title>
  		<link rel="stylesheet" href="/jboard2/css/style.css" />
  		<script	src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
  		<script src="/jboard2/js/listLoginCheck.js"></script>
  	</head>
  	<body>
  		<div id="board">
  			<h3>글목록</h3>
  			<!-- 리스트 -->
  			<div class="list">
  				<p class="logout">${ sessionScope.member.nick }님! 반갑습니다. <a href="/jboard2/member/logout.do">[로그아웃]</a><p>
  				<table>
  					<tr>
  						<td>번호</td>
  						<td>제목</td>
  						<td>글쓴이</td>
  						<td>날짜</td>
  						<td>조회</td>
  					</tr>
  					<c:forEach var="vo" items="${ list }">
  						<tr>
  							<td>${startNum}</td>
  							<!-- vo객체의 seq와 현재 페이지를 기반으로 데이터 요청 -->
  							<td><a href="/jboard2/view.do?seq=${vo.seq}&page=${page}" class="title">${vo.title}</a>&nbsp;[${vo.comment}]</td>
  							<td>${vo.nick}</td>
  							<td>${vo.rdate.substring(2,10)}</td>
  							<td>${vo.hit}</td>
  						</tr>
  						<c:set var="startNum" value="${startNum-1}" />
  					</c:forEach>
  				</table>
  			</div>
  			<!-- 페이징 -->
  			<nav class="paging">
  				<span>
  				<c:if test="${groupStart > 1}">
  					<a href="/jboard2/list.do?page=${groupStart-1}" class="prev">이전</a>
  				</c:if>
  				<a href="/jboard2/list.do?page=${prepage}"><</a>
  				<c:forEach var="i" begin="${groupStart}" end="${groupEnd}">
  					<a href="/jboard2/list.do?page=${i}" class="${i==page ? 'current':''} num">${i}</a>
  				</c:forEach>
  				<a href="/jboard2/list.do?page=${postpage}">></a>
  				<c:if test="${groupEnd < total_page }">
  					<a href="/jboard2/list.do?page=${groupEnd+1}" class="next">다음</a>
  				</c:if>

  				</span>
  			</nav>
  			<a href="/jboard2/write.do" class="btnWrite">글쓰기</a>
  		</div>
  	</body>
  </html>
  ```
- #### 글 보기(view.jsp)
  ```html
  <%@ page contentType="text/html;charset=UTF-8" pageEncoding="UTF-8"%>
  <!DOCTYPE html>
  <html>
  	<head>
  		<meta charset="UTF-8" />
  		<title>글쓰기</title>
  		<link rel="stylesheet" href="/jboard2/css/style.css" />
  	</head>
  	<body>
  		<div id="board">
  			<h3>글쓰기</h3>
  			<div class="write">
  				<form action="/jboard2/write.do" method="post">
  				<input type="hidden" name="cate" value="free"/>
  				<input type="hidden" name="uid" value="${member.uid }"/>
  					<table>
  						<tr>
  							<td>제목</td>
  							<td><input type="text" name="subject" placeholder="제목을 입력하세요." required /></td>
  						</tr>				
  						<tr>
  							<td>내용</td>
  							<td>
  								<textarea name="content" rows="20" required></textarea>
  							</td>
  						</tr>
  						<tr>
  							<td>첨부</td>
  							<td>
  								<input type="file" name="file" />
  							</td>
  						</tr>
  					</table>
  					<div class="btns">
  						<a href="/jboard2/list.do" class="cancel">취소</a>
  						<input type="submit" class="submit" value="작성완료" />
  					</div>
  				</form>
  			</div>
  		</div>
  	</body>
  </html>
  ```
- #### 글 쓰기(write.jsp)
  ```html
  <%@ page contentType="text/html;charset=UTF-8" pageEncoding="UTF-8"%>
  <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
  <!DOCTYPE html>
  <html>
  	<head>
  		<meta charset="UTF-8" />
  		<title>글보기</title>
  		<link rel="stylesheet" href="/jboard2/css/style.css" />
  	</head>
  	<body>
  		<div id="board">
  			<h3>글보기</h3>
  			<div class="view">
  				<form action="#" method="post">
  					<table>
  						<tr>
  							<td>제목</td>
  							<td>
  								<input type="text" name="subject" value="${vo.title}" readonly />
  							</td>
  						</tr>
  						<c:if test="${vo.file > 0}">
  						<tr>
  							<td>첨부파일</td>
  							<td>
  								<a href="#"></a>
  								<span></span>
  							</td>
  						</tr>
  						</c:if>
  						<tr>
  							<td>내용</td>
  							<td>
  								<textarea name="content" rows="20" readonly>${vo.contents}</textarea>
  							</td>
  						</tr>
  					</table>
  					<div class="btns">
  						<a href="/jboard2/delete.do?seq=${vo.seq}" class="cancel del">삭제</a>
  						<a href="#" class="cancel mod">수정</a>
  						<a href="/jboard2/list.do?page=${page}" class="cancel">목록</a>
  					</div>
  				</form>
  			</div><!-- view 끝 -->

  			<!-- 댓글리스트 -->
  			<section class="comments">
  				<h3>댓글목록</h3>
  				<c:forEach var="comment" items="${comments}">				
  					<div class="comment">
  						<span>
  							<span>${comment.uid}</span>
  							<span>${comment.rdate.substring(2,10) }</span>
  						</span>
  						<textarea>${comment.contents}</textarea>
  						<div>
  							<a href="#" class="del">삭제</a>
  							<a href="#" class="mod">수정</a>
  						</div>
  					</div>
  				</c:forEach>
  				<c:if test="${comments == '[]'}">				
  					<p class="empty">
  						등록된 댓글이 없습니다.
  					</p>
  				</c:if>

  			</section>

  			<!-- 댓글쓰기 -->
  			<section class="comment_write">
  				<h3>댓글쓰기</h3>
  				<div>
  					<form action="/jboard2/comment.do" method="post">
  						<input type="hidden" name="page" value="${page}" />
  						<input type="hidden" name="parent" value="${vo.seq}" />
  						<input type="hidden" name="cate" value="${vo.cate}" />
  						<input type="hidden" name="uid" value="${member.uid}" />
  						<textarea name="comment" rows="5"></textarea>
  						<div class="btns">
  							<a href="#" class="cancel">취소</a>
  							<input type="submit" class="submit" value="작성완료" />
  						</div>
  					</form>
  				</div>
  			</section>
  		</div><!-- board 끝 -->
  	</body>

  </html>
  ```

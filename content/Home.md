### 파이썬 코드
- github wiki 페이지에 있는 내 글들을 크롤링하여 데이터 베이스에 삽입하는 코드
```python
import requests as rq
from bs4 import BeautifulSoup
import pymysql
from urllib.request import urljoin
from slackclient import SlackClient

# soup 객체 리턴
def get_soup(url):
    res = rq.get(url)
    soup = BeautifulSoup(res.text, 'lxml')

    return soup

# 내 웹페이지 업데이트 - wiki
def my_git_crawler():
    # 메인 페이지
    base = "https://github.com/zeroam/studynote/wiki"

    soup = get_soup(base)

    # 메인 페이지에서 목록 가져오기
    titles = soup.select('li.Box-row a.d-block')

    # MySQL Connection 연결
    conn = pymysql.connect(host='zeroam.iptime.org', user='jcw', password='1234',
                           db='myblog', charset='utf8')

    #리턴값
    result = ""

    # 목록을 기반으로 url 만들기
    for tit_tag in titles:
        # 제목
        tit = tit_tag.text.strip()
        # a 태그의 href 값을 이용한 경로
        href = tit_tag.attrs['href']
        # 상대 경로 값 절대 경로로 바꾸기
        url = urljoin(base, href)

        # [] 안의 부분과 아닌 부분 나누기
        divided = tit.split(']')
        # [] 영역이 있는 글만 파일 생성
        if(len(divided) > 1):
            folder_name = divided[0][1:]
            file_name = divided[1]
            # 컨텐츠 추가
            # 중복 검사
            with conn.cursor() as curs:
                sql = "SELECT * FROM CONTENT WHERE cate=%s AND title=%s"
                value = (folder_name, file_name)
                curs.execute(sql, value)
                rs = curs.fetchall()

            # 결과 값이 없으면 카테고리 추가
            if(len(rs) == 0):
                soup = get_soup(url)
                body = soup.find('div', class_='markdown-body')
                with conn.cursor() as curs:
                    sql = "INSERT INTO CONTENT (cate, title, cont, rdate) VALUES (%s, %s, %s, NOW())"
                    value = (folder_name, file_name, str(body))
                    curs.execute(sql, value)
                conn.commit()
                result += tit + " -추가\n"
    # MySQL 접속 끊기
    conn.close()
    return result

# slack 메시지 전달
def notification(slackClient, message):
    slackClient.api_call(
            'chat.postMessage',
            channel='#general',
            text=message
    )

if __name__ == "__main__":
    # slack 토큰 값
    slack_token = '개인 slack 토큰 코드'

    # 크롤러 실행
    message = my_git_crawler()

    if message != "":
        # 토큰으로 slack 접속
        sc = SlackClient(slack_token)
        notification(sc, message +"\n업데이트 완료")
```

<br/>
<br/>

### 자바코드
- 해당 쿼리 값을 받아서 데이터 베이스에 있는 값 불러들임
- SQL 문
```sql
SELECT_LIST = "SELECT cate, title FROM CONTENT WHERE cate=?";
SELECT_CONTENT = "SELECT * FROM CONTENT WHERE cate=? AND title=?";
SELECT_DISTINCT_CATE = "SELECT DISTINCT cate FROM CONTENT";
```
- BlogDAO.java
```java
public class BlogDAO {

	// 싱글톤 객체
	private static BlogDAO dao = new BlogDAO();

	public static BlogDAO getInstance() {
		return dao;
	}

	private BlogDAO() {
	}

	// 드라이버 준비
	private Connection conn = null;
	private Statement stmt = null;
	private PreparedStatement psmt = null;
	private ResultSet rs = null;

	// 카테고리 가져오기
	public ArrayList<String> getCate() {
		ArrayList<String> cate = new ArrayList<>();
		String li = null;
		try {
			conn = DBConfig.getConnection();
			stmt = conn.createStatement();
			rs = stmt.executeQuery(SQL.SELECT_DISTINCT_CATE);

			while (rs.next()) {
				li = rs.getString(1);
				cate.add(li);
			}

		} catch (ClassNotFoundException | SQLException e) {
			e.printStackTrace();
		} finally {
			DBConfig.close(rs);
			DBConfig.close(stmt);
			DBConfig.close(conn);
		}
		return cate;
	}

	// 카테고리에 따른 리스트 가져오기
	public ArrayList<ContentVO> getList(String cate) {

		ArrayList<ContentVO> voList = new ArrayList<>();
		ContentVO vo = null;
		try {
			conn = DBConfig.getConnection();

			psmt = conn.prepareStatement(SQL.SELECT_LIST);
			psmt.setString(1, cate);

			rs = psmt.executeQuery();

			while (rs.next()) {
				vo = new ContentVO();
				vo.setCate(rs.getString(1));
				vo.setTitle(rs.getString(2));

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

	// 글 가져 오기
	public ContentVO getContent(String cate, String title) {
		ContentVO vo = null;
		try {
			conn = DBConfig.getConnection();

			psmt = conn.prepareStatement(SQL.SELECT_CONTENT);
			psmt.setString(1, cate);
			psmt.setString(2, title);

			rs = psmt.executeQuery();

			if (rs.next()) {
				vo = new ContentVO();
				vo.setSeq(rs.getInt(1));
				vo.setCate(rs.getString(2));
				vo.setTitle(rs.getString(3));
				vo.setcont(rs.getString(4));
				vo.setRdate(rs.getString(5));
			}
		} catch (ClassNotFoundException | SQLException e) {
			e.printStackTrace();
		} finally {
			DBConfig.close(rs);
			DBConfig.close(psmt);
			DBConfig.close(conn);
		}

		return vo;
	}
}
```
- BlogService.java
  - Map 객체에 ContentVO의 List 객체를 요소로 넣어 카테고리에 따른 리스트 출력하도록 구현
```java
public class BlogService implements CommandAction {

	@Override
	public String requestProc(HttpServletRequest req, HttpServletResponse resp) {

		String cate = req.getParameter("cate");
		String title = req.getParameter("title");

		Map<String, List<ContentVO>> cateMap = new HashMap<>();

		BlogDAO dao = BlogDAO.getInstance();
		// 카테고리 목록 가져오기
		List<String> cateList = dao.getCate();

		// 카테고리에 따른 리스트 목록 가져오기
		for (String str : cateList) {
			List<ContentVO> list = dao.getList(str);
			cateMap.put(str, list);
		}

		if (cate != null) {
			req.setAttribute("cate", cate);
		}

		// 컨텐츠 가져오기
		if (cate != null && title != null) {
			ContentVO content = dao.getContent(cate, title);
			req.setAttribute("content", content);
			req.setAttribute("title", title);
		}

		// 리퀘스트 속성 값 부여
		req.setAttribute("list", cateMap);

		return "blog.jsp";
	}
}
```

<br/>
<br/>

### JSP 코드(뷰 영역)
```html
<script>
	$(function() {

		var menu = $('div.col-lg-3 > div > a.list-group-item');

		menu.click(function(e) {
			e.preventDefault();

			var ul = $(this).next();
			var status = ul.is(':visible');

			if (status) {
				ul.slideUp(400);
				$(this).removeClass('active');
			} else {
				ul.slideDown(400);
				$(this).addClass('active');
			}
		});
	});
</script>
<c:if test="${content.cont != null }">
  <h2>
    <strong>${content.title}</strong>
  </h2>
  <hr />
  <br />
  ${content.cont}
</c:if>
```

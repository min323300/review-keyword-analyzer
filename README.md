# app.py
# Flask 웹 애플리케이션

from flask import Flask, render_template, request, jsonify, send_file
import os

from keyword_extractor import extract_keywords
from title_generator import generate_titles
from excel_exporter import export_to_excel, get_saved_files

app = Flask(__name__)
app.config['JSON_AS_ASCII'] = False


@app.route('/')
def index():
    """메인 페이지"""
    return render_template('index.html')


@app.route('/analyze', methods=['POST'])
def analyze():
    """리뷰 분석 API"""
    try:
        data = request.get_json()
        review_text = data.get('review', '').strip()
        num_keywords = int(data.get('num_keywords', 10))
        num_titles = int(data.get('num_titles', 5))
        save_excel = data.get('save_excel', True)
        
        if not review_text:
            return jsonify({'error': '리뷰 내용을 입력해주세요.'}), 400
        
        # 1. 키워드 추출
        keywords = extract_keywords(review_text, top_n=num_keywords)
        
        if not keywords:
            return jsonify({'error': '키워드를 추출할 수 없습니다.'}), 400
        
        keyword_list = [kw for kw, _ in keywords]
        
        # 2. 제목 생성
        result = generate_titles(keyword_list, count=num_titles)
        
        # 3. 엑셀 저장
        filepath = None
        if save_excel:
            filepath = export_to_excel(
                keywords=keywords,
                titles=result['titles'],
                category=result['category'],
                review_text=review_text
            )
        
        # 카테고리 한글명
        category_names = {
            'general': '일반',
            'restaurant': '맛집/음식',
            'medical': '의료/병원',
            'product': '제품/상품',
            'place': '장소/여행'
        }
        
        return jsonify({
            'success': True,
            'keywords': [{'word': kw, 'count': cnt} for kw, cnt in keywords],
            'category': result['category'],
            'category_name': category_names.get(result['category'], result['category']),
            'titles': result['titles'],
            'filepath': filepath
        })
        
    except Exception as e:
        return jsonify({'error': str(e)}), 500


@app.route('/files')
def files():
    """저장된 파일 목록 조회"""
    try:
        file_list = get_saved_files()
        return jsonify({'files': file_list})
    except Exception as e:
        return jsonify({'error': str(e)}), 500


@app.route('/download/<filename>')
def download(filename):
    """파일 다운로드"""
    try:
        filepath = os.path.join('output', filename)
        if os.path.exists(filepath):
            return send_file(filepath, as_attachment=True)
        return jsonify({'error': '파일을 찾을 수 없습니다.'}), 404
    except Exception as e:
        return jsonify({'error': str(e)}), 500


if __name__ == '__main__':
    # output 디렉토리 생성
    if not os.path.exists('output'):
        os.makedirs('output')
    
    print("\n" + "=" * 50)
    print("  📝 블로그 키워드 추출 & 제목 생성기")
    print("  http://localhost:5000 에서 실행 중...")
    print("=" * 50 + "\n")
    
    app.run(debug=True, host='0.0.0.0', port=5000)

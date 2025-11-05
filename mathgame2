import streamlit as st
import random
import time

# 페이지 설정
st.set_page_config(page_title="들이변환 테트리스", page_icon="🎮", layout="centered")

# 세션 상태 초기화
if 'board' not in st.session_state:
    st.session_state.board = [[0 for _ in range(10)] for _ in range(20)]
if 'score' not in st.session_state:
    st.session_state.score = 0
if 'current_question' not in st.session_state:
    st.session_state.current_question = None
if 'wrong_count' not in st.session_state:
    st.session_state.wrong_count = 0
if 'game_over' not in st.session_state:
    st.session_state.game_over = False
if 'block_ready' not in st.session_state:
    st.session_state.block_ready = False
if 'current_block' not in st.session_state:
    st.session_state.current_block = None
if 'block_position' not in st.session_state:
    st.session_state.block_position = 4

# 테트리스 블록 모양
BLOCKS = [
    [[1, 1, 1, 1]],  # I
    [[1, 1], [1, 1]],  # O
    [[0, 1, 0], [1, 1, 1]],  # T
    [[1, 1, 0], [0, 1, 1]],  # S
    [[0, 1, 1], [1, 1, 0]],  # Z
]

BLOCK_COLORS = ['🟦', '🟨', '🟪', '🟩', '🟥']

# 들이변환 문제 생성
def generate_question():
    question_types = [
        # L -> mL
        {'type': 'L_to_mL', 'value': random.randint(1, 5), 'unit': 'L', 'target': 'mL', 'multiply': 1000},
        # mL -> L (1000의 배수)
        {'type': 'mL_to_L', 'value': random.randint(1, 5) * 1000, 'unit': 'mL', 'target': 'L', 'multiply': 0.001},
        # dL -> mL
        {'type': 'dL_to_mL', 'value': random.randint(1, 9), 'unit': 'dL', 'target': 'mL', 'multiply': 100},
        # mL -> dL (100의 배수)
        {'type': 'mL_to_dL', 'value': random.randint(1, 9) * 100, 'unit': 'mL', 'target': 'dL', 'multiply': 0.01},
    ]
    
    q = random.choice(question_types)
    correct_answer = q['value'] * q['multiply']
    
    # 오답 생성
    if correct_answer >= 1:
        wrong_answers = [
            correct_answer * 10,
            correct_answer / 10,
            correct_answer + random.randint(1, 3)
        ]
    else:
        wrong_answers = [
            correct_answer * 10,
            correct_answer + 0.1,
            correct_answer + 0.01
        ]
    
    # 정답 형식 맞추기
    if correct_answer >= 1:
        correct_answer = int(correct_answer)
        wrong_answers = [int(w) if w >= 1 else w for w in wrong_answers]
    
    answers = [correct_answer] + wrong_answers[:3]
    random.shuffle(answers)
    
    return {
        'question': f"{q['value']}{q['unit']}는 몇 {q['target']}일까요?",
        'answers': answers,
        'correct': correct_answer
    }

# 블록 배치
def place_block(block, position):
    board = st.session_state.board
    block_height = len(block)
    block_width = len(block[0])
    
    # 블록을 맨 위부터 떨어뜨림
    row = 0
    while row < len(board) - block_height:
        can_place = True
        for i in range(block_height):
            for j in range(block_width):
                if block[i][j] == 1:
                    if board[row + i + 1][position + j] != 0:
                        can_place = False
                        break
            if not can_place:
                break
        if not can_place:
            break
        row += 1
    
    # 블록 배치
    for i in range(block_height):
        for j in range(block_width):
            if block[i][j] == 1:
                if row + i < len(board) and position + j < 10:
                    board[row + i][position + j] = 1
    
    st.session_state.board = board
    check_lines()

# 라인 체크 및 제거
def check_lines():
    board = st.session_state.board
    lines_cleared = 0
    
    new_board = []
    for row in board:
        if 0 in row:
            new_board.append(row)
        else:
            lines_cleared += 1
    
    # 제거된 라인만큼 위에 빈 라인 추가
    for _ in range(lines_cleared):
        new_board.insert(0, [0 for _ in range(10)])
    
    st.session_state.board = new_board
    st.session_state.score += lines_cleared * 100
    
    # 게임 오버 체크
    if any(st.session_state.board[0]):
        st.session_state.game_over = True

# 게임 리셋
def reset_game():
    st.session_state.board = [[0 for _ in range(10)] for _ in range(20)]
    st.session_state.score = 0
    st.session_state.wrong_count = 0
    st.session_state.game_over = False
    st.session_state.block_ready = False
    st.session_state.current_question = None
    st.session_state.current_block = None
    st.session_state.block_position = 4

# UI
st.title("🎮 들이변환 테트리스")
st.markdown("### 들이변환 문제를 풀고 블록을 쌓아보세요!")

# 상태 표시
col1, col2, col3 = st.columns(3)
with col1:
    st.metric("점수", st.session_state.score)
with col2:
    st.metric("틀린 횟수", f"{st.session_state.wrong_count}/3", 
              delta="게임오버!" if st.session_state.wrong_count >= 3 else None)
with col3:
    if st.button("🔄 새 게임"):
        reset_game()
        st.rerun()

st.markdown("---")

# 게임 오버 체크
if st.session_state.wrong_count >= 3:
    st.error("### 😢 3번 틀렸습니다! 다시 시작해주세요.")
    st.session_state.game_over = True
    if st.button("다시 시작하기", key="restart_wrong"):
        reset_game()
        st.rerun()
    st.stop()

if st.session_state.game_over:
    st.error("### 😢 게임 오버! 블록이 꽉 찼어요!")
    if st.button("다시 시작하기", key="restart_over"):
        reset_game()
        st.rerun()
    st.stop()

# 새 블록 생성
if not st.session_state.block_ready:
    st.session_state.current_question = generate_question()
    st.session_state.current_block = random.choice(BLOCKS)
    st.session_state.block_position = 4
    st.session_state.block_ready = True

# 문제 표시
if st.session_state.current_question:
    st.info(f"### 📝 {st.session_state.current_question['question']}")
    
    # 답안 버튼
    cols = st.columns(4)
    for idx, answer in enumerate(st.session_state.current_question['answers']):
        with cols[idx]:
            if st.button(f"{answer}", key=f"answer_{idx}", use_container_width=True):
                if answer == st.session_state.current_question['correct']:
                    st.success("🎉 정답입니다!")
                    time.sleep(0.5)
                    place_block(st.session_state.current_block, st.session_state.block_position)
                    st.session_state.block_ready = False
                    st.rerun()
                else:
                    st.session_state.wrong_count += 1
                    st.error(f"❌ 틀렸어요! ({st.session_state.wrong_count}/3)")
                    time.sleep(1)
                    st.rerun()

st.markdown("---")

# 블록 위치 조정
st.markdown("### 🎯 블록 위치 선택")
position_cols = st.columns([1, 3, 1])
with position_cols[0]:
    if st.button("⬅️ 왼쪽", use_container_width=True):
        if st.session_state.block_position > 0:
            block_width = len(st.session_state.current_block[0])
            if st.session_state.block_position > 0:
                st.session_state.block_position -= 1
                st.rerun()

with position_cols[1]:
    st.markdown(f"<div style='text-align: center; padding: 10px; background-color: #f0f0f0; border-radius: 5px;'>현재 위치: {st.session_state.block_position + 1}</div>", unsafe_allow_html=True)

with position_cols[2]:
    if st.button("➡️ 오른쪽", use_container_width=True):
        block_width = len(st.session_state.current_block[0])
        if st.session_state.block_position + block_width < 10:
            st.session_state.block_position += 1
            st.rerun()

st.markdown("---")

# 게임판 표시
st.markdown("### 🎲 게임판")
board_html = "<div style='font-family: monospace; font-size: 20px; line-height: 24px; background-color: #1a1a2e; padding: 10px; border-radius: 10px;'>"

for row_idx, row in enumerate(st.session_state.board):
    board_html += "<div style='display: flex; justify-content: center;'>"
    for col_idx, cell in enumerate(row):
        # 현재 블록 미리보기
        show_preview = False
        if st.session_state.current_block and row_idx < len(st.session_state.current_block):
            block_col = col_idx - st.session_state.block_position
            if 0 <= block_col < len(st.session_state.current_block[0]):
                if st.session_state.current_block[row_idx][block_col] == 1 and cell == 0:
                    show_preview = True
        
        if cell == 1:
            board_html += "🟦"
        elif show_preview:
            board_html += "⬜"
        else:
            board_html += "⬛"
    board_html += "</div>"

board_html += "</div>"
st.markdown(board_html, unsafe_allow_html=True)

# 도움말
with st.expander("📖 게임 방법"):
    st.markdown("""
    1. **들이변환 문제**를 풀어보세요! (L, dL, mL 변환)
    2. 정답을 맞추면 **블록이 떨어져요**!
    3. **왼쪽/오른쪽 버튼**으로 블록 위치를 조절하세요.
    4. 한 줄이 꽉 차면 **점수 100점**을 얻어요!
    5. **3번 틀리면** 게임이 끝나요. 조심하세요!
    
    **들이 단위 팁:**
    - 1L = 10dL = 1000mL
    - 1dL = 100mL
    """)

// app.js
import express from 'express';
import { Sequelize, DataTypes } from 'sequelize';
import bodyParser from 'body-parser';

// 1. DB 설정 (SQLite)
const sequelize = new Sequelize({
  dialect: 'sqlite',
  storage: 'database.sqlite',
  logging: false,
});

// 2. 모델 정의 (Domain/Entity)
const Member = sequelize.define('Member', {
  name: {
    type: DataTypes.STRING,
    allowNull: false,
  },
});

// 3. DTO 클래스
class MemberDto {
  constructor(name) {
    this.name = name;
  }
}

// 4. Repository
const MemberRepository = {
  save: async (memberEntity) => {
    return await Member.create(memberEntity);
  },
  findAll: async () => {
    return await Member.findAll();
  },
};

// 5. Service
const MemberService = {
  join: async (memberDto) => {
    const entity = { name: memberDto.name };
    return await MemberRepository.save(entity);
  },
  getAll: async () => {
    return await MemberRepository.findAll();
  },
};

// 6. Controller (Express 라우터)
const app = express();
app.use(bodyParser.json());

app.post('/members', async (req, res) => {
  const { name } = req.body;
  const dto = new MemberDto(name);
  const result = await MemberService.join(dto);
  res.json(result);
});

app.get('/members', async (req, res) => {
  const members = await MemberService.getAll();
  res.json(members);
});

// 7. 서버 시작
sequelize.sync().then(() => {
  console.log('DB 연결 완료!');
  app.listen(3000, () => {
    console.log('서버 실행 중: http://localhost:3000');
  });
});

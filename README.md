# 사용자를 기반으로 한 책 추천 알고리즘
---

1. 데이터 전처리 과정
    1. 데이터 읽기
    ``` python
        book_df=pd.read_csv('Books.csv',low_memory=False)
        rating_df=pd.read_csv('Ratings.csv')
        user_df=pd.read_csv('Users.csv')
    ```
    
    2. null 값 및 잘못된 형식의 값 확인
    ``` python
        book_df.isnull().sum()
        rating_df.isnull().sum()
        book_df.loc[book_df['Book-Author'].isnull(),:]
        book_df.loc[book_df['Publisher'].isnull(),:]
        book_df.loc[book_df['Image-URL-L'].isnull(),:]
    ```
    3. 정해진 형식의 값으로 채우기
    ``` python
        book_df['Book-Author']= book_df['Book-Author'].fillna('Other')
        book_df['Publisher']= book_df['Publisher'].fillna('Other')
        book_df['Image-URL-L']= book_df['Image-URL-L'].fillna('no_image.jpg')

        book_df.at[209538 ,'Publisher'] = 'DK Publishing Inc'
        book_df.at[209538 ,'Year-Of-Publication'] = 2000
        book_df.at[209538 ,'Book-Title'] = 'DK Readers: Creating the X-Men, How It All Began (Level 4: Proficient Readers)'
        book_df.at[209538 ,'Book-Author'] = 'Michael Teitelbaum'

        book_df.at[221678 ,'Publisher'] = 'DK Publishing Inc'
        book_df.at[221678 ,'Year-Of-Publication'] = 2000
        book_df.at[209538 ,'Book-Title'] = 'DK Readers: Creating the X-Men, How Comic book_df Come to Life (Level 4: Proficient Readers)'
        book_df.at[209538 ,'Book-Author'] = 'James Buckley'

        book_df.at[220731 ,'Publisher'] = 'Gallimard'
        book_df.at[220731 ,'Year-Of-Publication'] = '2003'
        book_df.at[209538 ,'Book-Title'] = 'Peuple du ciel - Suivi de Les bergers '
        book_df.at[209538 ,'Book-Author'] = 'Jean-Marie Gustave Le ClÃ?Â©zio'
    ```

    4. 데이터 병합
    ``` python
        dataset = pd.merge(book_df, rating_df, on='ISBN', how='inner')
        dataset = pd.merge(dataset, user_df, on='User-ID', how='inner')
        real_rating_dataset = dataset[dataset['Book-Rating'] != 0]
        real_rating_dataset = real_rating_dataset.reset_index(drop = True)
    ```

    5. 필요 없는 값 제외 및 정제
    ``` python
        df = real_rating_dataset.copy()
        df.dropna(inplace=True)
        df.reset_index(drop=True,inplace=True)
        df.drop(columns=["Year-Of-Publication","Image-URL-S","Image-URL-M"],axis=1,inplace=True)
        df.drop(index=df[df["Book-Rating"]==0].index,inplace=True)
        df["Book-Title"]=df["Book-Title"].apply(lambda x: re.sub("[\W_]+"," ",x).strip())
    ```
              
2. svd를 이용한 점수 예측 프로그램 설정
    1. reader 및 data 설정
    ``` python
        reader=Reader(rating_scale=(1,10))
        data=Dataset.load_from_df(df[['User-ID', 'ISBN', 'Book-Rating']], reader=reader)
    ```
    2. 데이터 학습
    ``` python
        trainset = data.build_full_trainset()
        svd.fit(trainset)
    ```
    3. 원하는 유저의 평가 항목 검색
    ```python
        # 원하는 유저의 평가 항목 검색
        # 아이디 입력를 하면 된다
        print('유저 리스트 입니다 다른 항목을 보려면 아무키나 누르세요 멈추 시려면 y를 누르세요')
        ma='a'
        while(ma!='y'):
            s=random.randint(1,len(df['User-ID']))
            print(df['User-ID'][s:s+10,])
            ma=input()

        id_1=input('원하는 유저의 아이디를 입력하세요: ')
        isbn=df.loc[df['User-ID']==int(id_1),'ISBN']
        ti=df.loc[df['User-ID']==int(id_1),'Book-Title']
        print('\n해당 유저가 평가한 책들의 ISBN과 제목입니다\n')
        print(isbn, ti)
        ISBN=input('평가를 원하시는 책의 isbn을 입력해주세요:')
        
    ```
        유저 리스트 입니다 다른 항목을 보려면 아무키나 누르세요 멈추 시려면 y를 누르세요
        190268    227250
        190269    227250
        190270    227250
        190271    227250
        190272    227250
        190273    227250
        190274    227250
        190275    227250
        190276    227250
        190277    227250
        Name: User-ID, dtype: int64
        227250
        264764    232528
        264765    232528
        264766    232528
        264767    166878
        264768    190758
        264769    235507
        264770    252802
        264771    190787
        264772    200527
        264773     61749
        Name: User-ID, dtype: int64
        y
        원하는 유저의 아이디를 입력하세요: 232528

        해당 유저가 평가한 책들의 ISBN과 제목입니다

        264764    8401462231
        264765    8459912019
        264766    8401499895

    4. 값 예측
    ``` python
        # 출력된 리스트 중에서 예상 점수를 확인하고자 하는 책의 ISBN 선택 후 문자열로 삽입
        print(f'평가 점수: {svd.predict(int(id_1),ISBN).est:.2f}')   
    ```
    평가를 원하시는 책의 isbn을 입력해주세요:0380820447
    평가 점수: 7.99

3. cosine 유사도를 이용한 책 추천 프로그램
    1. 추가적인 데이터 전처리 과정
    ```python
        new_df=df2[df2['User-ID'].map(df2['User-ID'].value_counts()) > 200]  # Drop users who vote less than 200 times.
        users_pivot=new_df.pivot_table(index=["User-ID"],columns=["Book-Title"],values="Book-Rating")
        users_pivot.fillna(0,inplace=True)
        new_df["User-ID"].values
        l=new_df["User-ID"].values
        idxa=random.randint(1,len(l))
    ```

    2. 필요 함수 설정
    ```python
            def users_choice(id):
    
                users_fav=new_df[new_df["User-ID"]==id].sort_values(["Book-Rating"],ascending=False)[0:5]
                return users_fav

            def user_based(new_df,id):
                if id not in new_df["User-ID"].values:
                    print("❌ User NOT FOUND ❌")
        
        
                else:
                    index=np.where(users_pivot.index==id)[0][0]
                    similarity=cosine_similarity(users_pivot)
                    similar_users=list(enumerate(similarity[index]))
                    similar_users = sorted(similar_users,key = lambda x:x[1],reverse=True)[0:5]
    
                     user_rec=[]
    
                    for i in similar_users:
                        data=df[df["User-ID"]==users_pivot.index[i[0]]]
                        user_rec.extend(list(data.drop_duplicates("User-ID")["User-ID"].values))

                return user_rec
            def common(new_df,user,user_id):
                x=new_df[new_df["User-ID"]==user_id]
                recommend_books=[]
                user=list(user)
                for i in user:
                    y=new_df[(new_df["User-ID"]==i)]
                    books=y.loc[~y["Book-Title"].isin(x["Book-Title"]),:]
                    books=books.sort_values(["Book-Rating"],ascending=False)[0:5]
                    recommend_books.extend(books["Book-Title"].values)
            
                return recommend_books[0:5]
    ```
    3. 원하는 유저의 평가 항목 검색
    ```python
        # 원하는 유저의 평가 항목 검색
        # 아이디 입력를 하면 된다
        print('유저 리스트 입니다 다른 항목을 보려면 아무키나 누르세요 멈추 시려면 y를 누르세요')
        ma='a'
        while(ma!='y'):
            s=random.randint(1,len(df['User-ID']))
            print(df['User-ID'][s:s+10,])
            ma=input()

        idxa=input('원하는 유저의 아이디를 입력하세요: ')
    ```
            유저 리스트 입니다 다른 항목을 보려면 아무키나 누르세요 멈추 시려면 y를 누르세요
            167839    44190
            167840    44190
            167841    44190
            167842    44190
            167843    44190
            167844    44190
            167845    44190
            167846    44190
            167847    44190
            167848    44190
            Name: User-ID, dtype: int64
            y
            원하는 유저의 아이디를 입력하세요: 31556
    4. 선호 하는 책과 그와 유사한 책들을 추천하고 이미지 링크 제공
    ```python
            user_id=l[idxa]
            user_choice_df=pd.DataFrame(users_choice(user_id))
            user_favorite=users_choice(user_id)
            n=len(user_choice_df["Book-Title"].values)
            print("USER: {} ".format(user_id))
            print()
                
            print("랜덤 선택된 유저가 선호하는 책")
            print()

            for i in range(n):
                b_name=new_df.loc[new_df["Book-Title"]==user_choice_df["Book-Title"].tolist()[i],"Book-Title"][:1].values[0]
                b_url=new_df.loc[new_df["Book-Title"]==user_choice_df["Book-Title"].tolist()[i],"Image-URL-L"][:1].values[0]
                print(f'제목 {b_name}')
                print(f'표지: {b_url}\n')
    ```
    4. 출력 결과
            USER: 31556 

            선택된 유저가 선호하는 책

            제목 To Kill a Mockingbird
            표지: http://images.amazon.com/images/P/0446310786.01.LZZZZZZZ.jpg

            제목 My Evil Twin An Avon Camelot Book
            표지: http://images.amazon.com/images/P/0380790823.01.LZZZZZZZ.jpg

            제목 A Wrinkle in Time
            표지: http://images.amazon.com/images/P/0440998050.01.LZZZZZZZ.jpg

            제목 Endurance Shackleton s Incredible Voyage
            표지: http://images.amazon.com/images/P/078670621X.01.LZZZZZZZ.jpg

            제목 We Live in Ireland Living Here
            표지: http://images.amazon.com/images/P/0531180700.01.LZZZZZZZ.jpg




            선호도 기반한 추천 책

            제목 Yukon Ho
            표지: http://images.amazon.com/images/P/0836218353.01.LZZZZZZZ.jpg

            제목 The Velveteen Rabbit
            표지: http://images.amazon.com/images/P/0380002558.01.LZZZZZZZ.jpg

            제목 Gulliver s Travels Dover Thrift Editions
            표지: http://images.amazon.com/images/P/0486292738.01.LZZZZZZZ.jpg

            제목 Cloudy With a Chance of Meatballs
            표지: http://images.amazon.com/images/P/0689707495.01.LZZZZZZZ.jpg

            제목 The Return of the Native
            표지: http://images.amazon.com/images/P/0451524713.01.LZZZZZZZ.jpg

reference
- <https://www.kaggle.com/datasets/arashnic/book-recommendation-dataset/data>
- <https://www.kaggle.com/code/scratchpad/notebook4060d37743/edit>
- <https://www.kaggle.com/code/hilalmleykeyuksel/book-recommender/notebook>
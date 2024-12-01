ng new video-platform
cd video-platform
npm install @angular/fire firebase angularx-social-login json-server
ng generate component login
ng generate component home
ng generate component video-page
ng generate component favorites
ng generate component search
ng generate component header
export const environment = {
  production: false,
  firebaseConfig: {
    apiKey: 'SUA_API_KEY',
    authDomain: 'SEU_AUTH_DOMAIN',
    projectId: 'SEU_PROJECT_ID',
    storageBucket: 'SEU_STORAGE_BUCKET',
    messagingSenderId: 'SEU_MESSAGING_SENDER_ID',
    appId: 'SEU_APP_ID',
    measurementId: 'SEU_MEASUREMENT_ID'
  }
};
import { Injectable } from '@angular/core';
import { AngularFireAuth } from '@angular/fire/auth';
import { GoogleAuthProvider, FacebookAuthProvider } from 'firebase/auth';
import firebase from 'firebase/compat/app';

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  constructor(private afAuth: AngularFireAuth) { }

  // Autenticação com Google
  loginWithGoogle() {
    return this.afAuth.signInWithPopup(new GoogleAuthProvider());
  }

  // Autenticação com Facebook
  loginWithFacebook() {
    return this.afAuth.signInWithPopup(new FacebookAuthProvider());
  }

  // Logout
  logout() {
    return this.afAuth.signOut();
  }
}
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { LoginComponent } from './login/login.component';
import { HomeComponent } from './home/home.component';
import { VideoPageComponent } from './video-page/video-page.component';
import { FavoritesComponent } from './favorites/favorites.component';

const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'login', component: LoginComponent },
  { path: 'video/:id', component: VideoPageComponent },
  { path: 'favorites', component: FavoritesComponent }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
 import { Component } from '@angular/core';
import { AuthService } from '../auth.service';
import { Router } from '@angular/router';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  styleUrls: ['./login.component.css']
})
export class LoginComponent {

  constructor(private authService: AuthService, private router: Router) { }

  loginWithGoogle() {
    this.authService.loginWithGoogle()
      .then(() => this.router.navigate(['/']))
      .catch(err => console.error(err));
  }

  loginWithFacebook() {
    this.authService.loginWithFacebook()
      .then(() => this.router.navigate(['/']))
      .catch(err => console.error(err));
  }
}
<div class="login-container">
  <button (click)="loginWithGoogle()">Login com Google</button>
  <button (click)="loginWithFacebook()">Login com Facebook</button>
</div>
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-home',
  templateUrl: './home.component.html',
  styleUrls: ['./home.component.css']
})
export class HomeComponent implements OnInit {

  videos = [
    { id: 1, title: 'Vídeo 1', description: 'Descrição do vídeo 1', thumbnail: 'url-para-miniatura' },
    { id: 2, title: 'Vídeo 2', description: 'Descrição do vídeo 2', thumbnail: 'url-para-miniatura' }
    // Mais vídeos aqui
  ];

  constructor() { }

  ngOnInit(): void { }

}
<div class="video-card" *ngFor="let video of videos">
  <img [src]="video.thumbnail" alt="Vídeo">
  <h3>{{ video.title }}</h3>
  <p>{{ video.description }}</p>
  <button [routerLink]="['/video', video.id]">Assistir</button>
</div>
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-video-page',
  templateUrl: './video-page.component.html',
  styleUrls: ['./video-page.component.css']
})
export class VideoPageComponent implements OnInit {

  video: any;

  constructor(private route: ActivatedRoute) { }

  ngOnInit(): void {
    const videoId = this.route.snapshot.paramMap.get('id');
    // Carregar vídeo e dados detalhados usando videoId
    this.video = { id: videoId, title: 'Exemplo de Vídeo', description: 'Descrição detalhada do vídeo', views: 1000 };
  }

  addView() {
    this.video.views++;
  }
}
<div class="video-container">
  <h1>{{ video.title }}</h1>
  <p>{{ video.description }}</p>
  <video controls>
    <source src="url-do-video.mp4" type="video/mp4">
  </video>
  <p>{{ video.views }} visualizações</p>
  <button (click)="addView()">Adicionar visualização</button>
</div>
{
  "videos": [
    { "id": 1, "title": "Vídeo 1", "description": "Descrição do vídeo 1", "views": 100 },
    { "id": 2, "title": "Vídeo 2", "description": "Descrição do vídeo 2", "views": 150 }
  ],
  "users": []
}
json-server --watch db.json --port 3000
export const environment = {
  production: false,
  firebaseConfig: {
    apiKey: 'SUA_API_KEY',
    authDomain: 'SEU_AUTH_DOMAIN',
    projectId: 'SEU_PROJECT_ID',
    storageBucket: 'SEU_STORAGE_BUCKET',
    messagingSenderId: 'SEU_MESSAGING_SENDER_ID',
    appId: 'SEU_APP_ID',
    measurementId: 'SEU_MEASUREMENT_ID'
  }
};
ng generate service history
import { Injectable } from '@angular/core';
import { AngularFirestore } from '@angular/fire/firestore';
import { AuthService } from './auth.service';
import firebase from 'firebase/compat/app';
import 'firebase/compat/firestore';

@Injectable({
  providedIn: 'root'
})
export class HistoryService {

  constructor(private firestore: AngularFirestore, private authService: AuthService) { }

  // Adicionar vídeo ao histórico de visualizações
  addVideoToHistory(videoId: string) {
    const user = this.authService.getCurrentUser();
    if (user) {
      const userHistoryRef = this.firestore.collection('userHistory').doc(user.uid).collection('history');
      return userHistoryRef.add({
        videoId: videoId,
        timestamp: firebase.firestore.FieldValue.serverTimestamp()
      });
    }
  }

  // Recuperar histórico de vídeos
  getUserHistory() {
    const user = this.authService.getCurrentUser();
    if (user) {
      return this.firestore.collection('userHistory').doc(user.uid).collection('history', ref => ref.orderBy('timestamp', 'desc')).valueChanges();
    }
  }
}
ng generate component history
import { Component, OnInit } from '@angular/core';
import { HistoryService } from '../history.service';

@Component({
  selector: 'app-history',
  templateUrl: './history.component.html',
  styleUrls: ['./history.component.css']
})
export class HistoryComponent implements OnInit {

  history: any[] = [];

  constructor(private historyService: HistoryService) { }

  ngOnInit(): void {
    this.historyService.getUserHistory().subscribe(historyData => {
      this.history = historyData;
    });
  }
}
<div class="history-container">
  <h1>Histórico de Visualizações</h1>
  <div *ngFor="let video of history">
    <p>Video ID: {{ video.videoId }} | Assistido em: {{ video.timestamp | date:'short' }}</p>
  </div>
</div>
